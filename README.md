

## Azure Active Directory を理解する
**対象：** 
   - オンプレミス Active Directory の動作は既に理解している人
   - クラウド (AzureAD) の知識は 0 <br />
   - これからクラウド (AzureAD) の利用を検討している人

**目的：** 
   - Azure AD の動作を理解する
   - Azure AD の利用方法を知る

<br /><br /><br />
## 理論編
- AzureAD を利用する利点
- セキュリティの考え方の変化



<br /><br /><br />
## 実装編

- [利用する認証オプションを理解する](AzureAD/AuthenticationOptions.md)
   - クラウドのみ
   - パスワードハッシュ同期 (PHS)
   - パススルー認証 (PTA)
   - フェデレーション (ADFS)


- 認証（認可) プロトコル
   - WS-Federation
   - OAuth/OpenID Connect
   - SAML

- Conditional Access



<br /><br /><br />
## (シナリオ編) AzureAD の導入/AzureAD への移行

- クラウドのみ
   - 既存オンプレミス環境なし ⇒ AzureAD を導入

- ハイブリッド環境 (ADFS なし）
   - シングルフォレスト・シングルドメイン ⇒ AzureAD ハイブリッド環境
   - シングルフォレスト・マルチドメイン ⇒ AzureAD ハイブリッド環境
   - マルチフォレスト ⇒ AzureAD ハイブリッド環境
   
- ハイブリッド環境 (ADFS あり）
   - シングルフォレスト・シングルドメイン ⇒ AzureAD ハイブリッド環境
   - シングルフォレスト・マルチドメイン ⇒ AzureAD ハイブリッド環境
   - マルチフォレスト ⇒ AzureAD ハイブリッド環境



<br /><br /><br />
## (手順編) 
- O365 ＋ AzureAD (クラウドのみ) のセットアップ
- 既存オンプレミス環境に O365 ＋ AzureAD (ハイブリッド環境) のセットアップ
- [検証環境：ADFS](StepByStep-ADFSTestEnvironment.md)


<br /><br /><br /><br /><br /><br />
###### [memo: Syntax](https://github.com/gentarom/Memo/blob/master/Syntax/README.md)
