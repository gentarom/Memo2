# Password Hash Authentication で同期する AzureAD-オンプレ環境を作る手順
Password Hash Authentication で同期する AzureAD-オンプレ環境を作る手順


## 1) ドメインの取得
外部からの名前解決が必要なのでドメインを取得する必要あり。  
検証目的なので .work や .xyz などの 1 円や 25 円でとれるドメインで OK
    
**注意：** 2年目以降は値段が上がるので、検証目的なら自動更新しない設定にしておくこと

## 2) オンプレミス環境の準備
オンプレミス環境を作成。ADFS 周りはあとでセットアップするので、とりあえず DC と ADCS 意外は OS をインストトールしてドメイン参加するところまででOK.

- ドメイン コントローラー 
  - 役割：Active Directory ドメイン サービス
  - 取得したドメイン名を使う
- Sync Server
  - ドメイン参加：する  
  - ドメイン内部との通信が必要
- クライアント
  - ドメイン参加：する
  - ドメイン内部との通信が必要

**注意：** OS によって制限あり。基本的には Windows Server 2016 以上であれば問題なし

## 3) AzureAD 環境の準備
Azure AD の無償テナントに登録する。
O365 と一緒に AzureAD が作れる以下が便利 （リンク切れの場合には 「O365 E5 無償」とかで検索)


  - Office 365 E5 https://products.office.com/ja-jp/business/office-365-enterprise-e5-business-software

 <kbd><img src="../master/Images/StepByStep-ADFSTestEnvironment002.jpg" width="80%" border="1"></kbd>

作ったアカウントで以下 2 つのポータルにサインインできることを確認。
- Azure ポータル [https://portal.azure.com](https://portal.azure.com)
- Office ポータル [https://portal.office.com](https://portal.office.com)

## 4) 作った AzureAD にカスタムドメイン名を登録
Azure ポータル (portal.azure.com) にサインインして、「Azure Active Directory」 - 「カスタム ドメイン名」- 「カスタム ドメイン名の追加」からカスタムドメインを追加。  

ドメインを取得したサイトから、ドメインの DNS に指定された TXT レコードを追加。(反映にしばらく時間がかかる。)  　　
「確認」が取れたらカスタムドメインが追加される~~ので、それを選択して 「プライマリにする」~~。

これでカスタムドメインを使用したユーザーを作成できる  
(User@xxx.onmicrosoft.com じゃなくて user@gentoso.com のユーザーを作ってサインインできるようになる)



## 5) Azure AD Connect の設定 
ADFS サーバーにログオンして、以下から Azure AD Connect のインストーラーをダウンロードして実行
  https://www.microsoft.com/en-us/download/details.aspx?id=47594

Microsoft Azure Active Directory Connect インストール ウィザード（各ステップの詳細は[ここ](https://docs.microsoft.com/ja-jp/azure/active-directory/hybrid/how-to-connect-install-custom)）  
既定の設定から変更する場所は↓ぐらい。基本的にウィザードに従って進めていくだけ。

- 簡単設定：「カスタマイズ」を選択
- ユーザー サインイン：「パスワード ハッシュの同期」 を選択

## 6) 動作確認
- Azure ポータル [https://portal.azure](https://portal.azure) にサインインし、「Azure Active Directory」 -「ユーザー」 にオンプレミス AD のユーザーが作成されていることを確認。
- Azure ポータル [https://portal.azure](https://portal.azure) にオンプレミス AD で作成したユーザーでサインインできることを確認。






## 動作確認



## 5) SSL 証明書を取得
今回は検証環境なのでドメインの CA から発行する。
- ADCS サーバーで certtmpl.msc (「証明テンプレート」コンソール) を開く
- 「証明書テンプレート」から 「Web サーバー」を右クリックして 「テンプレートの複製」
- 「新しいテンプレートのプロパティ」の以下を変更
    - 全般タブ：「テンプレート表示名」と「テンプレート名」を任意の名前に変更
    - 要求処理タブ：「秘密キーのエクスポートを許可する」をチェック
    - セキュリティタブ：Domain Computers に 登録 を許可
- certsrv.msc (「証明機関」コンソール) を開く
- 「証明書テンプレート」を右クリック、「新規作成」- 「発行する証明書テンプレート」から↑で作った証明書テンプレートを選択

- ADFS サーバーにログオンし、certlm.msc (「証明書」コンソール) を開く
- 「個人」を右クリック、「すべてのタスク」-「新しい証明書の要求」
- 「証明書の登録」ウィザードを進めて、作成した証明書テンプレートを選択。以下の情報を追加して、登録。
    - サブジェクト名： 「共通名」 - sts.<ドメイン名>
    - 別名： 「DNS」 - sts.<ドメイン名>
- 追加された証明書を右クリック、「すべたのタスク」-「エクスポート」。
    - はい、秘密キーをエクスポートします
    - .pfx 形式
    - 任意のパスワードとファイル名を指定

- DNS に sts.<ドメイン名>　が ADFS サーバーの IP アドレスに解決されるように A レコードを追加する
    


