---
title: Azure Event Grid から HTTP エンドポイントへのイベントの受信
description: HTTP エンドポイントを検証した後、Azure Event Grid からのイベントを受信して逆シリアル化する方法について説明します
services: event-grid
author: banisadr
manager: darosa
ms.service: event-grid
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: babanisa
ms.openlocfilehash: 89d0f11ccfb9a359ca3e43bc1a370e0fb7514574
ms.sourcegitcommit: 688a394c4901590bbcf5351f9afdf9e8f0c89505
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/18/2018
---
# <a name="receive-events-to-an-http-endpoint"></a>HTTP エンドポイントへのイベントの受信

この記事では、イベント サブスクリプションからイベントを受信する [HTTP エンドポイントを検証](security-authentication.md#webhook-event-delivery)した後、イベントを逆シリアル化する方法を説明します。 この記事では、デモンストレーション用に Azure 関数を使用しますが、アプリケーションがどこでホストされている場合でも、同じ概念を適用できます。

> [!NOTE]
> Event Grid で Azure 関数をトリガーするときは、[Event Grid トリガー](../azure-functions/functions-bindings-event-grid.md)を使用することを**強く**お勧めします。 ここでの汎用 WebHook トリガーの使用はデモンストレーションが目的です。

## <a name="prerequisites"></a>前提条件

* [HTTP によってトリガーされる関数](../azure-functions/functions-create-generic-webhook-triggered-function.md)を含む関数アプリが必要です

## <a name="add-dependencies"></a>依存関係を追加する

.NET で開発する場合は、`Microsoft.Azure.EventGrid` [Nuget パッケージ](https://www.nuget.org/packages/Microsoft.Azure.EventGrid)の関数に[依存関係を追加します](../azure-functions/functions-reference-csharp.md#referencing-custom-assemblies)。 他の言語用の SDK は、[発行 SDK](./sdk-overview.md#data-plane-sdks) リファレンスを介して利用できます。 これらのパッケージには、`EventGridEvent``StorageBlobCreatedEventData`、`EventHubCaptureFileCreatedEventData` などのネイティブなイベントの種類用のモデルが含まれています。

Azure 関数 (Azure functions ポータルの右端のウィンドウ) の [ファイルの表示] リンクをクリックし、project.json という名前のファイルを作成します。 `project.json` ファイルに次の内容を追加し、保存します。

 ```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.Azure.EventGrid": "1.1.0-preview"
      }
    }
   }
}
```

![追加された NuGet パッケージ](./media/receive-events/add-dependencies.png)

## <a name="endpoint-validation"></a>エンドポイントの検証

最初に実行することは、`Microsoft.EventGrid.SubscriptionValidationEvent` イベントの処理です。 だれかがイベントにサブスクライブするたびに、Event Grid は、データ ペイロード内に `validationCode` を含む検証イベントをエンドポイントに送信します。 エンドポイントは、これを応答本文にエコー バックして、[エンドポイントが有効であり、ユーザーによって所有されていることを証明する](security-authentication.md#webhook-event-delivery)必要があります。 WebHook によってトリガーされる関数ではなく [Event Grid トリガー](../azure-functions/functions-bindings-event-grid.md)を使用している場合、エンドポイントの検証は自動的に処理されます。 サード パーティ製 API サービス ([Zapier](https://zapier.com) や [IFTTT](https://ifttt.com/) など) を使用する場合は、検証コードをプログラムでエコーできないことがあります。 このようなサービスの場合は、サブスクリプション検証イベントで送信される検証 URL を使用すると、サブスクリプションを手動で検証できます。 その URL を `validationUrl` プロパティにコピーし、REST クライアントまたは Web ブラウザーのいずれかを使って GET 要求を送信します。

手動検証はプレビュー段階にあります。 使用するには、[AZ CLI 2.0](/cli/azure/install-azure-cli) 向けの [Event Grid 拡張](/cli/azure/azure-cli-extensions-list)をインストールする必要があります。 `az extension add --name eventgrid` でインストールできます。 REST API を使用している場合には、`api-version=2018-05-01-preview` を使用してください。

検証コードをプログラムでエコーするには、次のコードを使用します。

```csharp
using System.Net;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using Newtonsoft.Json.Serialization;
using Microsoft.Azure.EventGrid.Models;

class SubscriptionValidationEventData
{
    public string ValidationCode { get; set; }
}

class SubscriptionValidationResponseData
{
    public string ValidationResponse { get; set; }
}

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{

    log.Info($"C# HTTP trigger function begun");
    string response = string.Empty;
    const string SubscriptionValidationEvent = "Microsoft.EventGrid.SubscriptionValidationEvent";

    string requestContent = await req.Content.ReadAsStringAsync();
    EventGridEvent[] eventGridEvents = JsonConvert.DeserializeObject<EventGridEvent[]>(requestContent);

    foreach (EventGridEvent eventGridEvent in eventGridEvents)
    {
        JObject dataObject = eventGridEvent.Data as JObject;

        // Deserialize the event data into the appropriate type based on event type
        if (string.Equals(eventGridEvent.EventType, SubscriptionValidationEvent, StringComparison.OrdinalIgnoreCase))
        {
            var eventData = dataObject.ToObject<SubscriptionValidationEventData>();
            log.Info($"Got SubscriptionValidation event data, validation code: {eventData.ValidationCode}, topic: {eventGridEvent.Topic}");
            // Do any additional validation (as required) and then return back the below response
            var responseData = new SubscriptionValidationResponseData();
            responseData.ValidationResponse = eventData.ValidationCode;
            return req.CreateResponse(HttpStatusCode.OK, responseData);
        }
    }

    return req.CreateResponse(HttpStatusCode.OK, response);
}

```

```javascript
var http = require('http');

module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function begun');
    var validationEventType = "Microsoft.EventGrid.SubscriptionValidationEvent";

    for (var events in req.body) {
        var body = req.body[events];
        // Deserialize the event data into the appropriate type based on event type
        if (body.data && body.eventType == validationEventType) {
            context.log("Got SubscriptionValidation event data, validation code: " + body.data.validationCode + " topic: " + body.topic);

            // Do any additional validation (as required) and then return back the below response
            var code = body.data.validationCode;
            context.res = { status: 200, body: { "ValidationResponse": code } };
        }
    }
    context.done();
};
```

### <a name="test-validation-response"></a>検証の応答をテストする

サンプル イベントを関数のテスト フィールドに貼り付けることによって、検証応答関数をテストします。

```json
[{
  "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
  "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "subject": "",
  "data": {
    "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
  },
  "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
  "eventTime": "2018-01-25T22:12:19.4556811Z",
  "metadataVersion": "1",
  "dataVersion": "1"
}]
```

[実行] をクリックすると、200 OK と、本文に `{"ValidationResponse":"512d38b6-c7b8-40c8-89fe-f46f9e9622b6"}` が出力されます。

![検証の応答](./media/receive-events/validation-response.png)

## <a name="handle-blob-storage-events"></a>BLOB ストレージ イベントを処理する

次に、`Microsoft.Storage.BlobCreated` を処理するように関数を拡張してみましょう。

```cs
using System.Net;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using Newtonsoft.Json.Serialization;
using Microsoft.Azure.EventGrid.Models;

class SubscriptionValidationEventData
{
    public string ValidationCode { get; set; }
}

class SubscriptionValidationResponseData
{
    public string ValidationResponse { get; set; }
}

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info($"C# HTTP trigger function begun");
    string response = string.Empty;
    const string SubscriptionValidationEvent = "Microsoft.EventGrid.SubscriptionValidationEvent";
    const string StorageBlobCreatedEvent = "Microsoft.Storage.BlobCreated";

    string requestContent = await req.Content.ReadAsStringAsync();
    EventGridEvent[] eventGridEvents = JsonConvert.DeserializeObject<EventGridEvent[]>(requestContent);

    foreach (EventGridEvent eventGridEvent in eventGridEvents)
    {
        JObject dataObject = eventGridEvent.Data as JObject;

        // Deserialize the event data into the appropriate type based on event type 
        if (string.Equals(eventGridEvent.EventType, SubscriptionValidationEvent, StringComparison.OrdinalIgnoreCase))
        {
            var eventData = dataObject.ToObject<SubscriptionValidationEventData>();
            log.Info($"Got SubscriptionValidation event data, validation code: {eventData.ValidationCode}, topic: {eventGridEvent.Topic}");

            // Do any additional validation (as required) and then return back the below response
            var responseData = new SubscriptionValidationResponseData();
            responseData.ValidationResponse = eventData.ValidationCode;
            return req.CreateResponse(HttpStatusCode.OK, responseData);
        }

        else if (string.Equals(eventGridEvent.EventType, StorageBlobCreatedEvent, StringComparison.OrdinalIgnoreCase))
        {
            var eventData = dataObject.ToObject<StorageBlobCreatedEventData>();
            log.Info($"Got BlobCreated event data, blob URI {eventData.Url}");
        }
    }

    return req.CreateResponse(HttpStatusCode.OK, response);
}

```

```javascript
var http = require('http');

module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function begun');
    var validationEventType = "Microsoft.EventGrid.SubscriptionValidationEvent";
    var storageBlobCreatedEvent = "Microsoft.Storage.BlobCreated";

    for (var events in req.body) {
        var body = req.body[events];
        // Deserialize the event data into the appropriate type based on event type  
        if (body.data && body.eventType == validationEventType) {
            context.log("Got SubscriptionValidation event data, validation code: " + body.data.validationCode + " topic: " + body.topic);

            // Do any additional validation (as required) and then return back the below response
            var code = body.data.validationCode;
            context.res = { status: 200, body: { "ValidationResponse": code } };
        }

        else if (body.data && body.eventType == storageBlobCreatedEvent) {
            var blobCreatedEventData = body.data;
            context.log("Relaying received blob created event payload:" + JSON.stringify(blobCreatedEventData));
        }
    }
    context.done();
};

```

### <a name="test-blob-created-event-handling"></a>イベント処理用に作成した BLOB をテストする

テスト フィールドに [BLOB ストレージ イベント](./event-schema-blob-storage.md#example-event)を配置して実行することで、関数の新しい機能をテストします。

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/xstoretestaccount",
  "subject": "/blobServices/default/containers/testcontainer/blobs/testfile.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "0x8D4BCC2E4835CD0",
    "contentType": "text/plain",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://example.blob.core.windows.net/testcontainer/testfile.txt",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

関数ログに BLOB URL が出力されます。

![出力ログ](./media/receive-events/blob-event-response.png)

Blob ストレージ アカウントまたは General Purpose V2 (GPv2) Storage アカウントを作成し、[イベント サブスクリプションを追加](../storage/blobs/storage-blob-event-quickstart.md)し、エンドポイントを関数 URL に設定してテストすることもできます。

![関数の URL](./media/receive-events/function-url.png)

## <a name="handle-custom-events"></a>カスタム イベントを処理する

最後に、関数をもう一度拡張して、カスタム イベントも処理できるようにしましょう。 イベント `Contoso.Items.ItemReceived` のチェックを追加します。 最終的なコードは、次のようになります。

```cs
using System.Net;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using Newtonsoft.Json.Serialization;
using Microsoft.Azure.EventGrid.Models;

class SubscriptionValidationEventData
{
    public string ValidationCode { get; set; }
}

class SubscriptionValidationResponseData
{
    public string ValidationResponse { get; set; }
}

class ContosoItemReceivedEventData
{
    public string id { get; set; }
    public string message { get; set; }
    public string time { get; set; }
}

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info($"C# HTTP trigger function begun");
    string response = string.Empty;
    const string SubscriptionValidationEvent = "Microsoft.EventGrid.SubscriptionValidationEvent";
    const string StorageBlobCreatedEvent = "Microsoft.Storage.BlobCreated";
    const string CustomTopicEvent = "Contoso.Items.ItemReceived";

    string requestContent = await req.Content.ReadAsStringAsync();
    EventGridEvent[] eventGridEvents = JsonConvert.DeserializeObject<EventGridEvent[]>(requestContent);

    foreach (EventGridEvent eventGridEvent in eventGridEvents)
    {
        JObject dataObject = eventGridEvent.Data as JObject;

        // Deserialize the event data into the appropriate type based on event type
        if (string.Equals(eventGridEvent.EventType, SubscriptionValidationEvent, StringComparison.OrdinalIgnoreCase))
        {
            var eventData = dataObject.ToObject<SubscriptionValidationEventData>();
            log.Info($"Got SubscriptionValidation event data, validation code: {eventData.ValidationCode}, topic: {eventGridEvent.Topic}");
            // Do any additional validation (as required) and then return back the below response
            var responseData = new SubscriptionValidationResponseData();
            responseData.ValidationResponse = eventData.ValidationCode;
            return req.CreateResponse(HttpStatusCode.OK, responseData);
        }

        else if (string.Equals(eventGridEvent.EventType, StorageBlobCreatedEvent, StringComparison.OrdinalIgnoreCase))
        {
            var eventData = dataObject.ToObject<StorageBlobCreatedEventData>();
            log.Info($"Got BlobCreated event data, blob URI {eventData.Url}");
        }

        else if (string.Equals(eventGridEvent.EventType, CustomTopicEvent, StringComparison.OrdinalIgnoreCase))
        {
            var eventData = dataObject.ToObject<ContosoItemReceivedEventData>();
            log.Info($"Got ContosoItemReceived event data, item URI {eventData.id}");
        }
    }

    return req.CreateResponse(HttpStatusCode.OK, response);
}

```

```javascript
var http = require('http');
var t = require('tcomb');

module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function begun');
    var validationEventType = "Microsoft.EventGrid.SubscriptionValidationEvent";
    var storageBlobCreatedEvent = "Microsoft.Storage.BlobCreated";
    var customEventType = "Contoso.Items.ItemReceived";
    var ContosoItemReceivedEventData = t.struct({
        id: t.Str,
        message: t.Str,
        time: t.Str,
    })

    for (var events in req.body) {
        var body = req.body[events];
        // Deserialize the event data into the appropriate type based on event type
        if (body.data && body.eventType == validationEventType) {
            context.log("Got SubscriptionValidation event data, validation code: " + body.data.validationCode + " topic: " + body.topic);

            // Do any additional validation (as required) and then return back the below response
            var code = body.data.validationCode;
            context.res = { status: 200, body: { "ValidationResponse": code } };
        }

        else if (body.data && body.eventType == storageBlobCreatedEvent) {
            var blobCreatedEventData = body.data;
            context.log("Relaying received blob created event payload:" + JSON.stringify(blobCreatedEventData));
        }

        else if (body.data && body.eventType == customEventType) {
            var payload = new ContosoItemReceivedEventData(body.data);
            context.log("Relaying received custom payload:" + JSON.stringify(payload));
            context.log(payload instanceof ContosoItemReceivedEventData);
        }
    }
    context.done();
};

```

### <a name="test-custom-event-handling"></a>カスタム イベントの処理をテストする

最後に、拡張した関数がカスタム イベントの種類を処理できるようになっていることをテストします。

```json
[{
    "subject": "Contoso/foo/bar/items",
    "eventType": "Microsoft.EventGrid.CustomEventType",
    "eventTime": "2017-08-16T01:57:26.005121Z",
    "id": "602a88ef-0001-00e6-1233-1646070610ea",
    "data": { 
            "id": "831e1650-001e-001b-66ab-eeb76e069631",
            "message": "Contoso item Foo",
            "time": "2017-06-26T18:41:00.9584103Z"
            },
    "dataVersion": "",
    "metadataVersion": "1"
}]
```

この機能は、[カスタム イベントと CURL をポータルから送信する](./custom-event-quickstart-portal.md)か、[Postman](https://www.getpostman.com/) などのエンドポイントに POST できる任意のサービスまたはアプリケーションを使用して[カスタム トピックを投稿する](./post-to-custom-topic.md)ことで、ライブでテストすることもできます。 エンドポイントが関数 URL として設定されたカスタム トピックとイベント サブスクリプションを作成します。

## <a name="next-steps"></a>次の手順

* [Azure Event Grid の管理 SDK と発行 SDK](./sdk-overview.md) を確認する
* [カスタム トピックを投稿する](./post-to-custom-topic.md)方法を確認する
* [BLOB ストレージにアップロード済みの画像のサイズ変更](resize-images-on-storage-blob-upload-event.md)などの Event Grid と関数の詳細なチュートリアルを試す
