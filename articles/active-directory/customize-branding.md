---
title: Azure Active Directory テナントのサインイン ページをカスタマイズする | Microsoft Docs
description: Azure サインイン ページに会社のブランドを追加する方法について説明します。
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 01/19/2018
ms.author: curtand
ms.reviewer: kexia
custom: it-pro
ms.openlocfilehash: ac8562b77ba43dbb07e886dfa235586198027796
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/07/2018
ms.locfileid: "33764446"
---
# <a name="quickstart-add-company-branding-to-your-sign-in-page-in-azure-ad"></a>クイック スタート: Azure AD のサインイン ページに会社のブランドを追加する
多くの企業は、管理下にある Web サイトとサービスに関して利用者が戸惑わないよう、そのすべての外観に統一感を持たせたいと考えています。 Azure Active Directory (Azure AD) では、会社のロゴや独自の配色でサインイン ページの外観をカスタマイズできるようにすることでこれを実現します。 サインイン ページは、Azure AD を ID プロバイダーとして使用する、Office 365 などの Web ベースのアプリケーションにサインインするときに表示されます。 このページを使用して資格情報を入力します。

> [!NOTE]
> * 会社のブランドを使用できるのは、Azure AD の Premium または Basic ライセンスを購入した場合、または Office 365 のライセンスを所有している場合だけです。 ご利用のライセンス タイプで機能がサポートされているかどうかについては、[Azure Active Directory の価格情報](https://azure.microsoft.com/pricing/details/active-directory/)に関するページを参照してください。
> 
> * Azure Active Directory のワールドワイド インスタンスを使用している中国のお客様は、Azure AD Premium および Basic エディションを使用できます。 中国の 21Vianet が運営する Azure サービスでは、Azure AD Premium および Basic エディションは現在サポートされていません。 詳細については、[Azure Active Directory フォーラム](https://feedback.azure.com/forums/169401-azure-active-directory/)からお問い合わせください。

## <a name="customizing-the-sign-in-page"></a>サインイン ページのカスタマイズ

<!--You can customize the following elements on the sign-in page: <attach image>-->

カスタマイズした会社のブランドが Azure AD のサインイン ページに表示されるのは、ユーザーがテナント固有の URL ([*https://outlook.com/contoso.com*](https://outlook.com/contoso.com) など) にアクセスしたときです。

たとえば、ユーザーが www.office.com にアクセスしたときに表示されるサインイン ページには、カスタマイズした会社のブランドは表示されません。そのユーザーはまだ資格情報を入力していないためです。 ユーザーがユーザー ID を入力するか、ユーザー タイルを選択すると、会社のブランドが表示されます。

> [!NOTE]
> * ドメイン名は、ブランドを構成した Azure Portal の **[ドメイン]** 部分に "アクティブ" として表示される必要があります。 詳細については、[カスタム ドメイン名の追加](add-custom-domain.md)に関する記事をご覧ください。
> * サインイン ページのブランド情報が、個人用 Microsoft アカウントのサインイン ページに継承されることはありません。 つまり従業員やゲスト ユーザーが個人用 Microsoft アカウントでサインインした場合、そのサインイン ページには組織のブランドが表示されません。


### <a name="banner-logo"></a>バナー ロゴ 

[説明] | 制約 | Recommendations
------- | ------- | ----------
バナー ロゴはサインイン ページとアクセス パネル ページに表示されます。<br>サインイン ページでは、ロゴは、ユーザー名が入力された後に表示されます。 | 透過 JPG または透過 PNG<br>高さの最大値: 36 ピクセル<br>幅の最大値: 245 ピクセル | 組織のロゴはここで使用します。<br>透過画像を使用してください。 背景が白になることを前提にしないでください。<br>画像のロゴの周囲にパディングを追加しないでください。追加すると、ロゴが過度に小さくなります。

### <a name="username-hint"></a>ユーザー名のヒント   
[説明] | 制約 | Recommendations
------- | ------- | ----------
このオプションは、ユーザー名フィールドのヒント テキストをカスタマイズします。 | 最大 64 文字の Unicode テキスト<br>プレーン テキストのみ | 所属する組織の外部からゲスト ユーザーがアプリにサインインすることが予想される場合、このオプションは設定しないようにすることをお勧めします。
            
### <a name="sign-in-page-text"></a>サインイン ページ テキスト   
[説明] | 制約 | Recommendations
------- | ------- | ----------
このオプションは、サインイン フォームの下部に表示され、ヘルプ デスクの電話番号や法的声明などの追加情報を知らせるために使用できます。 | 最大 256 文字の Unicode テキスト<br>プレーン テキストのみ (リンクや HTML タグを含まない)    

### <a name="sign-in-page-image"></a>サインイン ページの画像  
[説明] | 制約 | Recommendations
------- | ------- | ----------
このオプションは、サインイン ページの背景に表示され、表示可能な領域の中央に固定されて、ブラウザー ウィンドウを埋めるように拡大縮小またはトリミングされます。    <br>携帯電話のように小さい画面では、この画像は表示されません。<br>ページが読み込まれるときに、不透明度 0.55 の黒いマスクがこの画像に適用されます。 | JPG または PNG<br>画像サイズ: 1920 x 1080 ピクセル<br>ファイル サイズ: &lt; 300 KB | <br>強調して表示したいものが集中していない場所でこの画像を使用します。 非透過のサインイン フォームがこの画像の中央に覆いかぶさるように表示され、ブラウザー ウィンドウのサイズに応じて画像の一部を覆うことができます。<br>ファイル サイズを小さいままにして、読み込み時間を短縮します。 

### <a name="sign-in-page-background-color"></a>サインイン ページの背景色
[説明] | 制約 | Recommendations
------- | ------- | ----------
回線速度が遅い場合に、この色が背景画像の代わりに表示されます。 | 16 進数の RGB 色 (例: #FFFFFF) | バナー ロゴまたは所属する組織のプライマリ カラーを使用することをお勧めします。

### <a name="square-logo-image"></a>正方形のロゴ イメージ
[説明] | 制約 | Recommendations
------- | ------- | ----------
この画像は、新しい Enterprise Windows 10 PC の設定中に表示されます。 これは、新しい作業用 PC を設定している従業員にコンテキストを提供します。 画像は、[Windows AutoPilot](https://blogs.windows.com/business/2017/06/29/delivering-modern-promise-windows-10/?utm_source=dlvr.it&utm_medium=twitter#gDTp1u6q35bvDWIS.97) を使用して作業デバイスをデプロイするテナントに対して表示されます。また、他の Windows 10 操作ではパスワード入力ページに表示されます。 | 透過 PNG (推奨) または JPG<br>画像サイズ: 240 x 240 ピクセル<br>ファイル サイズ: &lt; 10 KB | 組織のロゴはここで使用します。<br> 透過画像を使用してください。<br>背景が白になることを前提にしないでください。<br>画像のロゴにパディングを追加しないでください。追加すると、ロゴが過度に小さくなります。

### <a name="show-option-to-remain-signed-in"></a>サインインしたままにする表示オプション
[説明] | 制約 | Recommendations
------- | ------- | ----------
Azure AD サインインでは、ユーザーはブラウザーを閉じたり開き直したりしたときにサインインしたままの状態にするオプションを選択できます。 この設定は、そのオプションを非表示にします。<br>これを **[いいえ]** に設定すると、ユーザーにはこのオプションが表示されなくなります。 | &nbsp; | このオプションを非表示にしても、セッションの有効期間には影響しません。<br>SharePoint Online と Office 2010 の一部の機能は、ユーザーがサインインしたままにすることを選択できるかどうかに依存します。 このオプションを **[いいえ]** に設定すると、ユーザーにサインインを求める追加のメッセージが予期せず表示されることがあります。

> [!NOTE]
> すべての要素は省略できます。 たとえば、背景画像なしでバナー ロゴを指定した場合、サインイン ページにはロゴとともに目的のサイトの背景画像が表示されます (例: Office 365)。

## <a name="add-company-branding-to-your-directory"></a>ディレクトリに会社のブランドを追加するには

1. テナントのグローバル管理者のアカウントで [Azure AD 管理センター](https://aad.portal.azure.com)にサインインします。
2. **[ユーザーとグループ]** > **[会社のブランド]** > **[編集]** を選択します。
  
  ![カスタム ブランドを開く](./media/customize-branding/navigation-to-branding.png)
3. カスタマイズする要素に変更を加えます。 すべての要素は省略できます。
  
  ![カスタム ブランドを編集する](./media/customize-branding/edit-branding.png)
5. 終了したら、**[保存]** を選択します。

サインイン ページのブランドに加えた変更が表示されるまでに、最大 1 時間かかる場合があります。

## <a name="add-language-specific-company-branding-to-your-directory"></a>言語固有の会社のブランドをディレクトリに追加するには

1. ディレクトリのグローバル管理者のアカウントで [Azure AD 管理センター](https://aad.portal.azure.com)にサインインします。
2. **[ユーザーとグループ]** > **[会社のブランド]** > **[新しい言語]** を選択します。
  
  ![言語固有のブランド要素を追加する](./media/customize-branding/add-language.png)
5. カスタマイズする要素に変更を加えます。 すべての要素は省略できます。
6. 終了したら、**[保存]** を選択します。

サインイン ページのブランドに加えた変更が表示されるまでに、最大 1 時間かかる場合があります。

## <a name="next-steps"></a>次の手順
このクイック スタートでは、Azure AD ディレクトリに会社のブランドを追加する方法について学習しました。 

次のリンクを使用して、Azure ポータルから、会社のブランドを Azure AD に構成することができます。

> [!div class="nextstepaction"]
> [会社のブランドの構成](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/LoginTenantBrandingBlade) 