---
title: "定期的に実行されるワークフローとタスクのスケジューリング - Azure Logic Apps | Microsoft Docs"
description: "定期的に実行されるタスク、アクション、ワークフロー、プロセス、ワークロードをロジック アプリで作成し、スケジューリングします。"
services: logic-apps
documentationcenter: 
author: ecfan
manager: anneta
editor: 
tags: connectors
ms.assetid: 51dd4f22-7dc5-41af-a0a9-e7148378cd50
ms.service: logic-apps
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/25/2017
ms.author: LADocs; estfan
ms.openlocfilehash: 0dead955f9eb723dfa232d3ce751498a09ce1b29
ms.sourcegitcommit: 0b02e180f02ca3acbfb2f91ca3e36989df0f2d9c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2018
---
# <a name="create-and-schedule-regularly-running-tasks-with-azure-logic-apps"></a>Azure Logic Apps を使用して定期的に実行されるタスクを作成し、スケジュールを設定する

定期的に実行されるタスク、アクション、ワークロード、プロセスをスケジューリングするには、**[スケジュール - 繰り返し]** [トリガー](../logic-apps/logic-apps-overview.md#logic-app-concepts)で開始するロジック アプリ ワークフローを作成します。 このトリガーを使用すると、タスクの実行の繰り返しを開始する日時と繰り返しのスケジュールを設定できます。たとえば、次のような例があります。

* 内部データを取得する: 毎日 [SQL ストアド プロシージャを実行](../connectors/connectors-create-api-sqlazure.md)する。
* 外部データを取得する: NOAA から 15 分ごとに天気予報を取り込む。
* データを報告する: 過去 1 週間について特定の金額を超えるすべての注文の概要をメールする。
* データを処理する: 平日に毎日、ピーク時間帯を避けて、その日にアップロードした画像を圧縮する。
* データをクリーンアップする: 3 か月より前のツイートをすべて削除する。
* データをアーカイブする: 毎月バックアップ サービスに請求書をプッシュする。

このトリガーでは多数のパターンがサポートされます。たとえば:

* すぐに実行して、*n* 期間 (秒、分、時、日、週、月) ごとに繰り返す。
* 特定の時刻に開始して、*n* 期間 (秒、分、時、日、週、月) ごとに実行を繰り返す。
* 毎日 1 回または複数回実行する (8:00 AM と 5:00 PM など)。
* 毎週特定の曜日 (土曜日と日曜日など) に実行する。
* 毎週特定の曜日の特定の時刻 (月曜日から金曜日の 8:00 AM と 5:00 PM など) に実行する。

繰り返しトリガーが作動するたびに、ロジック アプリ ワークフローの新しいインスタンスが Logic Apps によって作成されて実行されます。

## <a name="prerequisites"></a>前提条件

* Azure サブスクリプション。 Azure サブスクリプションがない場合は、[無料の Azure アカウントで作業を開始](https://azure.microsoft.com/free/)できます。 また、[従量課金のサブスクリプションにサインアップ](https://azure.microsoft.com/pricing/purchase-options/)することもできます。

* [ロジック アプリの作成方法](../logic-apps/quickstart-create-first-logic-app-workflow.md)に関する基本的な知識 

## <a name="add-a-recurrence-trigger-to-your-logic-app"></a>ロジック アプリへの繰り返しトリガーの追加

1. [Azure Portal](https://portal.azure.com) にサインインします。 空のロジック アプリを作成します。[空のロジック アプリを作成する方法](../logic-apps/quickstart-create-first-logic-app-workflow.md)に関するページを参照してください。

2. Logic Apps デザイナーが表示されたら、検索ボックスにフィルターとして「繰り返し」と入力します。 **[スケジュール - 繰り返し]** トリガーを選択します。 

   ![[スケジュール - 繰り返し] トリガー](./media/connectors-native-recurrence/add-recurrence-trigger.png)

   このトリガーがロジック アプリの最初のステップになります。

3. 繰り返しの間隔と頻度を設定します。 この例では、これらのプロパティを設定して、ワークフローを毎週実行します。 

   ![間隔と頻度の設定](./media/connectors-native-recurrence/recurrence-trigger-details.png)

4. その他のスケジュール オプションを表示するには、**[詳細オプションを表示する]** を選択します。 

   ![その他のオプション](./media/connectors-native-recurrence/recurrence-trigger-more-options.png)

5. これで、次のオプションを設定することができます。 

   * トリガーを開始する日付と時刻を設定します。 
   開始する日付と時刻を指定する場合、タイム ゾーンも適用できます。 

   * 頻度に [日] または [週] を選択した場合は、繰り返しの具体的な時刻を選択できます。 

   * [週] を選択した場合は、特定の曜日も選択できます。
   
   ![詳細スケジュール オプション](./media/connectors-native-recurrence/recurrence-trigger-more-options-details.png)

   たとえば、今日が 2017 年 9 月 4 日の月曜日だとします。 
   次の繰り返しトリガーでは、作動するのは開始日時である 2017 年 9 月 18 日 (月曜日) 8:00 AM PST "*以降*" となります。 
   ただし、繰り返しのスケジュールは月曜日の 10:30 AM、12:30 PM、2:30 PM にのみ設定されています。 したがって、最初にトリガーが作動してロジック アプリのワークフロー インスタンスが作成される時刻は 10:30 AM となります。 
   開始時刻の動作について詳しくは、[開始時刻の例](#start-time)を参照してください。
   その後は、同じ日の 12:30 PM と 2:30 PM に実行されます。 
   繰り返しのたびに、ワークフロー インスタンスが新しく作成されます。 その後は、次の月曜日に再び、スケジュール全体が最初から繰り返されます。 
   "[*その他の繰り返しの例*](#example-recurrences)"

   ![詳細スケジュールの例](./media/connectors-native-recurrence/recurrence-trigger-more-options-advanced-schedule.png)

   > [!NOTE]
   > 指定した繰り返しのプレビューが表示されるのは、トリガーの頻度として [日] または [週] を選択したときだけです。
   
6. アクションまたはフロー制御ステートメントを使って、ワークフローを完成させます。 追加できるアクションの詳細については、[コネクタ](../connectors/apis-list.md)に関するページを参照してください。 

## <a name="trigger-details"></a>トリガーの詳細

繰り返しトリガーに関して構成できるプロパティは次のとおりです。

| Name | 必須 | プロパティ名 | type | [説明] | 
|----- | -------- | ------------- | ---- | ----------- | 
| **頻度** | [はい] | frequency | String | 繰り返しの時間の単位。**秒**、**分**、**時**、**日**、**週**、**月**のいずれかになります。 | 
| **間隔** | [はい] | interval | 整数 | ワークフローの実行間隔を、[頻度] に指定された単位に基づいて表す正の整数。 <p>既定の間隔は 1 です。 間隔の最小値と最大値は次のとおりです。 <p>- month: 1 ～ 16 か月 </br>- day: 1 ～ 500 日 </br>- hour: 1 ～ 12,000 時間 </br>- minute: 1 ～ 72,000 分 </br>- 秒: 1 ～ 9,999,999 秒<p>たとえば間隔が 6 で、頻度が "月" である場合は、繰り返しは 6 か月ごとになります。 | 
| **タイム ゾーン** | いいえ  | timeZone | String | 開始時刻を指定したときに限り適用されます。このトリガーに [UTC オフセット](https://en.wikipedia.org/wiki/UTC_offset)を指定することはできないためです。 適用するタイム ゾーンを選択してください。 | 
| **開始時刻** | いいえ  | startTime | String | 開始時刻を次の形式で指定します。 <p>YYYY-MM-DDThh:mm:ss (タイム ゾーンを選択した場合) <p>または <p>YYYY-MM-DDThh:mm:ssZ (タイム ゾーンを選択しなかった場合) <p>たとえば、2017 年 9 月 18 日の 2:00 PM とする場合、「2017-09-18T14:00:00」と指定し、太平洋標準時などのタイム ゾーンを選択します。 または、タイム ゾーンなしで「2017-09-18T14:00:00Z」と指定します。 <p>**注:** この開始時刻は、[UTC オフセット](https://en.wikipedia.org/wiki/UTC_offset)を除いた [UTC 日時形式](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)で、[日付と時刻に関する ISO 8601 規格](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations)に従って入力する必要があります。 タイム ゾーンを選択しない場合は、末尾にスペースを入れず、アルファベットの "Z" を追加してください。 この "Z" は、同等の[航海時間](https://en.wikipedia.org/wiki/Nautical_time)を表します。 <p>単純なスケジュールでは、開始時刻と最初の実行時刻が一致するのに対して、複雑なスケジュールでは、トリガーが作動するのは開始時刻以降となります。 "[*開始日時の使用方法を具体的に教えてください*](#start-time)" | 
| **設定曜日** | いいえ  | weekDays | 文字列または文字列配列 | [週] を選択した場合、**月曜日**、**火曜日**、**水曜日**、**木曜日**、**金曜日**、**土曜日**、**日曜日**の中から、ワークフローを実行する曜日を 1 つまたは複数選択できます。 | 
| **設定時刻 (時間)** | いいえ  | hours | 整数または整数配列 | [日] または [週] を選択した場合、ワークフローを実行する時刻 (時) として 0 ～ 23 の整数を選択できます (複数選択可)。 <p>たとえば "10"、"12"、"14" を指定した場合、時の要素として 10 AM、12 PM、2 PM が設定されます。 | 
| **設定時刻 (分)** | いいえ  | minutes | 整数または整数配列 | [日] または [週] を選択した場合、ワークフローを実行する時刻 (分) として 0 ～ 59 の整数を選択できます (複数選択可)。 <p>たとえば上の例で指定した時を使用し、分の要素に「30」を指定した場合、実行時刻は 10:30 AM、12:30 PM、2:30 PM となります。 | 
||||| 

## <a name="json-example"></a>JSON の例

[繰り返しトリガーの定義](../logic-apps/logic-apps-workflow-actions-triggers.md#recurrence-trigger)の例を次に示します。

``` json
{
    "triggers": {
        "Recurrence": {
            "type": "Recurrence",
            "recurrence": {
                "frequency": "Week",
                "interval": 1,
                "schedule": {
                    "hours": [
                        10,
                        12,
                        14
                    ],
                    "minutes": [
                        30
                    ],
                    "weekDays": [
                        "Monday"
                    ]
                },
               "startTime": "2017-09-07T14:00:00",
               "timeZone": "Pacific Standard Time"
            }
        }
    }
}
```

## <a name="faq"></a>FAQ

<a name="example-recurrences"></a>

**Q:** 他にも繰り返しのスケジュールの例があれば教えてください </br>
**A:** 以下、他にもいくつかの例を紹介します。

| 繰り返し | 間隔 | 頻度 | 開始時刻 | 設定曜日 | 設定時刻 (時間) | 設定時刻 (分) | 注 |
| ---------- | -------- | --------- | ---------- | ------------- | -------------- | ---------------- | ---- |
| 15 分ごとに実行 (開始日時は指定せず) | 15 | [分] | {なし} | {使用不可} | {なし} | {なし} | このスケジュールは直ちに開始され、以降、繰り返しのタイミングは前回の実行時刻に基づいて計算されます。 | 
| 15 分ごとに実行 (開始日時を指定) | 15 | [分] | *startDate*T*startTime*Z | {使用不可} | {なし} | {なし} | このスケジュールは、指定された開始日時 "*以降*" に開始され、その後、繰り返しのタイミングは前回の実行時刻に基づいて計算されます。 | 
| 1 時間ごとの正時に実行 (開始日時を指定) | 1 | Hour | *startDate*Thh:00:00Z | {使用不可} | {なし} | {なし} | このスケジュールは、指定された開始日時 "*以降*" に開始されます。 以降、繰り返しが実行されるのは毎時 "00" 分となります。 <p>頻度が [週] または [月] である場合、このスケジュールは 1 週間に 1 日だけ、または 1 か月に 1 日だけ実行されます。 | 
| 毎日 1 時間ごとに実行 (開始日時は指定せず) | 1 | Hour | {なし} | {使用不可} | {なし} | {なし} | このスケジュールは直ちに開始され、以降、繰り返しのタイミングは前回の実行時刻に基づいて計算されます。 <p>頻度が [週] または [月] である場合、このスケジュールは 1 週間に 1 日だけ、または 1 か月に 1 日だけ実行されます。 | 
| 毎日 1 時間ごとに実行 (開始日時を指定) | 1 | Hour | *startDate*T*startTime*Z | {使用不可} | {なし} | {なし} | このスケジュールは、指定された開始日時 "*以降*" に開始され、その後、繰り返しのタイミングは前回の実行時刻に基づいて計算されます。 <p>頻度が [週] または [月] である場合、このスケジュールは 1 週間に 1 日だけ、または 1 か月に 1 日だけ実行されます。 | 
| 毎正時 15 分に実行 (開始日時を指定) | 1 | Hour | *startDate*T00:15:00Z | {使用不可} | {なし} | {なし} | このスケジュールは、指定された開始日時 "*以降*" に開始され、00:15 AM、1:15 AM、2:15 AM などに実行されます。 | 
| 毎正時 15 分に実行 (開始日時は指定せず) | 1 | 日 | {なし} | {使用不可} | 0、1、2、3、4、5、6、7、8、9、10、11、12、13、14、15、16、17、18、19、20、21、22、23 | 15 | このスケジュールは 00:15 AM、1:15 AM、2:15 AM などに実行されます。 また、このスケジュールは、頻度を "時" に、開始時刻を "15" 分に設定することと等価です。 | 
| 15 分ごとに、15 分刻みで実行 (開始日時は指定せず) | 1 | 日 | {なし} | {使用不可} | 0、1、2、3、4、5、6、7、8、9、10、11、12、13、14、15、16、17、18、19、20、21、22、23 | 0、15、30、45 | このスケジュールは、次に来る 15 分刻みの時刻になるまで開始されません。 | 
| 毎日 8:00 AM に実行 (開始日時は指定せず) | 1 | 日 | {なし} | {使用不可} | 8 | {なし} | このスケジュールは毎日 8:00 AM に、指定されたスケジュールに基づいて実行されます。 | 
| 毎日 8:00 AM に実行 (開始日時を指定) | 1 | 日 | *startDate*T08:00:00Z | {使用不可} | {なし} | {なし} | このスケジュールは毎日 8:00 AM に、指定された開始時刻に基づいて実行されます。 | 
| 毎日 8:30 AM に実行 (開始日時は指定せず) | 1 | 日 | {なし} | {使用不可} | 8 | 30 | このスケジュールは毎日 8:30 AM に、指定されたスケジュールに基づいて実行されます。 | 
| 毎日 8:30 AM に実行 (開始日時を指定) | 1 | 日 | *startDate*T08:30:00Z | {使用不可} | {なし} | {なし} | このスケジュールは、指定された開始日の 8:30 AM に開始されます。 | 
| 毎日 8:30 AM と 4:30 PM に実行 | 1 | 日 | {なし} | {使用不可} | 8、16 | 30 | | 
| 毎日午前 8 時 30 分、午前 8 時 45 分、午後 4 時 30 分、午後 4 時 45 分に実行 | 1 | 日 | {なし} | {使用不可} | 8、16 | 30、45 | | 
| 毎週土曜日の 5:00 PM に実行 (開始日時は指定せず) | 1 | 週 | {なし} | "土曜日" | 17 | 00 | このスケジュールは毎週土曜日の 5:00 PM に実行されます。 | 
| 毎週土曜日の 5:00 PM に実行 (開始日時を指定) | 1 | 週 | *startDate*T17:00:00Z | "土曜日" | {なし} | {なし} | このスケジュールは、指定された開始日時 "*以降*" に開始されます。このケースでは 2017 年 9 月 9 日の 5:00 PM となります。 以降、繰り返しが実行されるのは毎週土曜日の 5:00 PM となります。 | 
| 毎週火曜日と木曜日の 5:00 PM に実行 | 1 | 週 | {なし} | "火曜日"、"木曜日" | 17 | {なし} | このスケジュールは毎週火曜日と木曜日の 5:00 PM に実行されます。 | 
| 業務時間中 1 時間ごとに実行 | 1 | 週 | {なし} | 土曜日と日曜日を除くすべての曜日を選択 | 時刻 (時) を選択 | 時刻 (分) を選択 | たとえば、業務時間が 8:00 AM ～ 5:00 PM である場合、時刻 (時) として "8、9、10、11、12、13、14、15、16、17" を選択します。 <p>業務時間が 8:30 AM ～ 5:30 PM である場合は、前述の時刻 (時) のほか、分として "30" を選択します。 | 
| 週末に毎日 1 回実行 | 1 | 週 | {なし} | "土曜日"、"日曜日" | 時刻 (時) を選択 | 時刻 (分) を選択 | このスケジュールは毎週土曜日と日曜日に、指定されたスケジュールで実行されます。 | 
| 隔週月曜日のみ 15 分間隔で実行 | 2 | 週 | {なし} | "月曜日" | 0、1、2、3、4、5、6、7、8、9、10、11、12、13、14、15、16、17、18、19、20、21、22、23 | 0、15、30、45 | このスケジュールは、隔週月曜日の 15 分刻みの時刻ごとに実行されます。 | 
| 1 か月に 1 日、1 時間ごとに実行 | 1 | 月 | {「注」を参照} | {使用不可} | 0、1、2、3、4、5、6、7、8、9、10、11、12、13、14、15、16、17、18、19、20、21、22、23 | {「注」を参照} | 開始日時を指定しなかった場合、このスケジュールでは作成日時が使用されます。 繰り返しのスケジュールの分を制御するには、開始時刻の分の要素を指定するか、作成時刻を使用します。 たとえば、開始時刻または作成時刻が 8:25 AM である場合、このスケジュールは 8:25 AM、9:25 AM、10:25 AM などに実行されます。 | 
||||||||| 

<a name="start-time"></a>

**Q:** 開始日時の使用方法を具体的に教えてください </br>
**A:** 以下、開始日時を使って繰り返しを制御する方法とその繰り返しが Logic Apps エンジンによって実行される方法について、いくつかのパターンを紹介します。

| 開始時刻 | スケジュールなしの繰り返し | スケジュールありの繰り返し | 
| ---------- | --------------------------- | ------------------------ | 
| {なし} | 初回ワークロードが即座に実行されます。 <p>それ以降は、前回の実行時刻に基づいてワークロードが実行されます。 | 初回ワークロードが即座に実行されます。 <p>それ以降は、指定されたスケジュールに基づいてワークロードが実行されます。 | 
| 開始時刻を過去に設定 | 実行時刻は、指定された開始時刻に基づいて計算され、過去の実行時刻については放棄されます。 初回ワークロードが実行されるのは、次の実行時刻となります。 <p>それ以降は、前回の実行時刻を基準とする計算に基づいてワークロードが実行されます。 <p>詳細については、この表の後に示した例を参照してください。 | 初回ワークロードは、開始時刻 "*以降*" に、その開始時刻から計算されたスケジュールに基づいて実行されます。 <p>それ以降は、指定されたスケジュールに基づいてワークロードが実行されます。 <p>**注:** スケジュール付きで繰り返しを指定した場合で、なおかつ時刻の時または分を指定しなかった場合、将来の実行時刻は、初回実行時刻を基準とした時または分を使用して計算されます。 | 
| 開始時刻を現在または将来に設定 | 初回ワークロードが実行されるのは、指定された開始時刻となります。 <p>それ以降は、前回の実行時刻を基準とする計算に基づいてワークロードが実行されます。 | 初回ワークロードは、開始時刻 "*以降*" に、その開始時刻から計算されたスケジュールに基づいて実行されます。 <p>それ以降は、指定されたスケジュールに基づいてワークロードが実行されます。 <p>**注:** スケジュール付きで繰り返しを指定した場合で、なおかつ時刻の時または分を指定しなかった場合、将来の実行時刻は、初回実行時刻を基準とした時または分を使用して計算されます。 | 
||||

**開始時刻が過去である例 (繰り返しの指定のみでスケジュールは指定しない)** 

| 開始時刻 | 現在の時刻 | 繰り返し | スケジュール |
| ---------- | ------------ | ---------- | -------- | 
| 2017-09-**07**T14:00:00Z | 2017-09-**08**T13:00:00Z | 2 日ごと | {なし} | 
||||| 

このシナリオでは、実行時刻は開始時刻に基づいて Logic Apps エンジンによって計算されます。過去の実行時刻は放棄され、初回実行には、次に訪れる開始時刻が使用されます。 初回実行の後、以降の実行には、開始時刻から計算されたスケジュールが使用されます。 この繰り返しは次のようになります。

| 開始時刻 | 初回実行時刻 | 将来実行時刻 | 
| ---------- | ------------ | ---------- | 
| 2017-09-**07** の 2:00 PM | 2017-09-**09** の 2:00 PM | 2017-09-**11** の 2:00 PM </br>2017-09-**13** の 2:00 PM </br>2017-09-**15** の 2:00 PM </br>以下同様
||||| 

したがってこのシナリオでは、開始時刻としてどれだけ古い日付 (2017-09-**05** の 2:00 PM や 2017-09-**01** の 2:00 PM など) を指定したとしても、初回実行時刻は変わりません。

## <a name="next-steps"></a>次の手順

* [ワークフローのアクションとトリガー](../logic-apps/logic-apps-workflow-actions-triggers.md#recurrence-trigger)
* [コネクタ](../connectors/apis-list.md)
