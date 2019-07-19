# ADFS 検証環境を作る手順

新規で ADFS の検証環境を立てるための手順

## 1) ドメインの取得
外部からの名前解決が必要なのでドメインを取得する必要あり。  
検証目的なので .work や .xyz などの 1 円や 25 円でとれるドメインで OK
    
**注意：** 2年目以降は値段が上がるので、検証目的なら自動更新しない設定にしておくこと

## 2) オンプレミス環境の準備
オンプレミス環境を作成。ADFS 周りはあとでセットアップするので、とりあえず DC と ADCS 意外は OS をインストトールしてドメイン参加するところまででOK.

- ドメイン コントローラー 
  - 役割：Active Directory ドメイン サービス
  - 取得したドメイン名を使う
- ADCS サーバー
  - 役割：Active Directory 証明書サービス
    - エンタープライズ CA
    - 暗号化プロバイダ：RSA＃ Microsoft Software Key Storage Provider
    - ハッシュ： SHA-256    
  - ドメイン参加：する
  - ドメイン内部との通信が必要
- ADFS サーバー
  - ドメイン参加：する
  - ドメイン内部との通信が必要
- Sync Server
  - ドメイン参加：する  
  - ドメイン内部との通信が必要
- ADFS Proxy Server
  - ドメイン参加：しない
  - ドメイン内部と外部 (インターネット/DMZ) との通信が必要
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
「確認」が取れたらカスタムドメインが追加されるので、それを選択して 「プライマリにする」。

これでカスタムドメインを使用したユーザーを作成できる  
(User@xxx.onmicrosoft.com じゃなくて user@gentoso.com のユーザーを作ってサインインできるようになる)

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
    

## 6) Azure AD Connect の設定 (ADFS/Proxy のインストール)
ADFS サーバーにログオンして、以下から Azure AD Connect のインストーラーをダウンロードして実行
  https://www.microsoft.com/en-us/download/details.aspx?id=47594

Microsoft Azure Active Directory Connect インストール ウィザード（各ステップの詳細は[ここ](https://docs.microsoft.com/ja-jp/azure/active-directory/hybrid/how-to-connect-install-custom)）  
既定の設定から変更する場所は↓ぐらい。基本的にウィザードに従って進めていくだけ。

- 簡単設定：「カスタマイズ」を選択
- ユーザー サインイン：「AD FS とのフェデレーション」 を選択して「次へ」
- AD FS ファーム：「新しい AD FS ファームを構成する」を選択し、↑でエクスポートしておいた「証明書ファイル」を選択
- AD FS サーバー：ADFS サーバーを指定
- Proxy サーバー：Proxy サーバーを指定。Proxy サーバーはドメインに参加していないので、色々設定が必要
    - 権限は Proxy サーバーのローカル管理者持っているユーザーを指定する
    - [必要なポート](https://docs.microsoft.com/ja-jp/azure/active-directory/hybrid/reference-connect-ports) がすべて空いていること (firewall でブロックされていないこと）
    - Proxy サーバーのWindows Remote Management (WS-Management) サービスが実行中
    - Proxy サーバーの Powershell (管理者) で Enable-PSRemoting –force が実行済み
    - 同期サーバーで Set-Item WSMan:\localhost\Client\TrustedHosts –Value <プロキシサーバーFQDN> -Force –Concatenate を実行  
      注意：FQDN 指定必須。ただし、Proxy サーバーがドメインに参加してないのでそのままだと FQDN 指定ができないので、DNS に Proxy サーバーの A レコードを追加して ProxyServer.gentoso.com で名前解決できるようにする。
    - Proxy サーバーが sts の SSL 証明書の発行元 (エンタープライズ CA) を信頼する。  
      ADFS サーバーで certlm.msc (「証明書」コンソール) を開いて、「信頼されたルート証明機関」にあるエンタープライズCAの証明書をエクスポート。このファイルを Proxy サーバーの certlm.msc の「信頼されたルート照明機関」にインポート。


## 動作確認
