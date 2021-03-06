---
title: PowerShell で Azure DNS の使用を開始する | Microsoft Docs
description: Azure DNS で、DNS ゾーンとレコードを作成する方法について説明します。 PowerShell を使用して最初の DNS ゾーンとレコードを作成して管理するためのステップ バイ ステップ ガイドです。
services: dns
documentationcenter: na
author: KumudD
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: fb0aa0a6-d096-4d6a-b2f6-eda1c64f6182
ms.service: dns
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/10/2017
ms.author: kumud
ms.openlocfilehash: 050111f4a5e8459e89d049ccb879b5079ff68527
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/23/2018
ms.locfileid: "30175326"
---
# <a name="get-started-with-azure-dns-using-powershell"></a>PowerShell で Azure DNS の使用を開始する

> [!div class="op_single_selector"]
> * [Azure Portal](dns-getstarted-portal.md)
> * [PowerShell](dns-getstarted-powershell.md)
> * [Azure CLI 1.0](dns-getstarted-cli-nodejs.md)
> * [Azure CLI 2.0](dns-getstarted-cli.md)

この記事では、Azure PowerShell を使用して最初の DNS ゾーンとレコードを作成する手順について説明します。 これらの手順は、Azure Portal またはクロスプラットフォームの Azure CLI を使用して実行することもできます。 Azure DNS は、プライベート ドメインの作成もサポートします。 プライベート DNS ゾーンとレコードを初めて作成する方法に関する順を追った説明については、「[PowerShell で Azure DNS プライベート ゾーンの使用を開始する](private-dns-getstarted-powershell.md)」を参照してください。

DNS ゾーンは、特定のドメインの DNS レコードをホストするために使用されます。 Azure DNS でドメインのホストを開始するには、そのドメイン名用に DNS ゾーンを作成する必要があります。 ドメインの DNS レコードはすべて、この DNS ゾーン内に作成されます。 最後に、DNS ゾーンをインターネットに公開するには、ドメインのネーム サーバーを構成する必要があります。 ここでは、その手順について説明します。

以降の手順は、Azure PowerShell がインストール済みで、既にサインインしていることを前提としています。 詳細については、「[PowerShell を使用して DNS ゾーンを管理する方法](dns-operations-dnszones.md)」を参照してください。

## <a name="create-the-resource-group"></a>リソース グループの作成

DNS ゾーンを作成する前に、DNS ゾーンが含まれるリソース グループを作成します。 コマンドを次に示します。

```powershell
New-AzureRMResourceGroup -name MyResourceGroup -location "westus"
```

## <a name="create-a-dns-zone"></a>DNS ゾーンの作成

DNS ゾーンは、 `New-AzureRmDnsZone` コマンドレットを使用して作成します。 次の例では、*MyResourceGroup* というリソース グループに *contoso.com* という DNS ゾーンを作成します。 この例の値を実際の値に置き換えて、DNS ゾーンを作成できます。

```powershell
New-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyResourceGroup
```
Azure DNS で新たにプライベート DNS ゾーンがサポートされました (現在はパブリック プレビュー)。  プライベート DNS ゾーンの詳細については、「[Using Azure DNS for private domains (プライベート ドメインでの Azure DNS の使用)](private-dns-overview.md)」をご覧ください。 プライベート DNS ゾーンを作成する例については、「[PowerShell で Azure DNS プライベート ゾーンの使用を開始する](./private-dns-getstarted-powershell.md)」を参照してください。

## <a name="create-a-dns-record"></a>DNS レコードの作成

レコード セットは、`New-AzureRmDnsRecordSet` コマンドレットを使用して作成します。 下の例では、リソース グループ "MyResourceGroup" で DNS ゾーン "contoso.com" に相対名 "www" を持つレコードを作成します。 レコード セットの完全修飾名は、"www.contoso.com" になります。 また、レコードの種類は "A"、IP アドレスは "1.2.3.4"、TTL は 3,600 秒です。

```powershell
New-AzureRmDnsRecordSet -Name www -RecordType A -ZoneName contoso.com -ResourceGroupName MyResourceGroup -Ttl 3600 -DnsRecords (New-AzureRmDnsRecordConfig -IPv4Address "1.2.3.4")
```

その他のレコードの種類、複数のレコードを持つレコード セット、既存のレコードの変更については、[Azure PowerShell を使用した DNS レコードおよびレコード セットの管理](dns-operations-recordsets.md)に関するページをご覧ください。 


## <a name="view-records"></a>レコードの表示

ゾーンで DNS レコードを表示するには、次を使用します。

```powershell
Get-AzureRmDnsRecordSet -ZoneName contoso.com -ResourceGroupName MyResourceGroup
```


## <a name="update-name-servers"></a>ネーム サーバーの更新

DNS ゾーンとレコードを正しく設定したら、Azure DNS ネーム サーバーを使用するようにドメイン名を構成する必要があります。 これにより、インターネット上の他のユーザーが DNS レコードを検索できるようになります。

ゾーンのネーム サーバーを指定するには、`Get-AzureRmDnsZone` コマンドレットを使用します。

```powershell
Get-AzureRmDnsZone -Name contoso.com -ResourceGroupName MyResourceGroup

Name                  : contoso.com
ResourceGroupName     : myresourcegroup
Etag                  : 00000003-0000-0000-b40d-0996b97ed101
Tags                  : {}
NameServers           : {ns1-01.azure-dns.com., ns2-01.azure-dns.net., ns3-01.azure-dns.org., ns4-01.azure-dns.info.}
NumberOfRecordSets    : 3
MaxNumberOfRecordSets : 5000
```

このネーム サーバーは、ドメイン名レジストラー (ドメイン名を購入した場所) で構成する必要があります。 レジストラーにより、ドメインのネーム サーバーを設定するオプションが提供されます。 詳細については、「[Azure DNS へのドメインの委任](dns-domain-delegation.md)」を参照してください。

## <a name="delete-all-resources"></a>すべてのリソースの削除

この記事で作成したすべてのリソースを削除するには、次の手順を実行します。

```powershell
Remove-AzureRMResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>次の手順

Azure DNS の詳細については、「[Azure DNS の概要](dns-overview.md)」を参照してください。

Azure DNS での DNS ゾーンの管理の詳細については、[PowerShell を使用した Azure DNS での DNS ゾーンの管理](dns-operations-dnszones.md)に関するページをご覧ください。

Azure DNS での DNS レコードの管理の詳細については、[PowerShell を使用した Azure DNS での DNS レコードおよびレコード セットの管理](dns-operations-recordsets.md)に関するページをご覧ください。

