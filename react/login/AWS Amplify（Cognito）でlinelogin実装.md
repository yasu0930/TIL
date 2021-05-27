# **AWS Amplify（Cognito）を使ったLINEログインの実装**
* ### **前提**
・Amplifyの設定が完了していること(amplify configure、amplify initおよびamplify pushが済んでいること)

・Amplifyで作成したreact.jsアプリが存在すること(aws-amplifyとaws-amplify-reactの導入・設定が済んでいること)

Amplifyの状態は以下のとおり（amplify status）
```php
$ amplify status

Current Environment: prod

| Category | Resource name | Operation | Provider plugin |
| -------- | ------------- | --------- | --------------- |
```

* ### **手順**

LINEとのログイン連携の手順は以下のとおりです。


1.LINE側（LINE Developers）の設定

2.認証モジュールの追加（amplify add auth）

3.CongnitoのLINE連携（OpenID Connect）設定

4.LINE側（LINE Developers）にリダイレクトURLの設定


* ### **1. LINE側（LINE Developers）の設定**


今回は、下記簡略に手順だけ記載する。
 ### **Step1. LINE Developers ( https://developers.line.biz/ja/ ） にログインします**
 ### **Step2. プロバイダーを作成します。**

---

* ### **2. 認証モジュールの追加（amplify add auth）**
下記、設定の全文となります。

```php
% amplify add auth
Using service: Cognito, provided by: awscloudformation

 The current configured provider is Amazon Cognito.

 Do you want to use the default authentication and security configuration? Default configuration with Social Provider (Federation)
 Warning: you will not be able to edit these selections.
 How do you want users to be able to sign in? Username
 Do you want to configure advanced settings? Yes, I want to make some additional changes.
 Warning: you will not be able to edit these selections.
 What attributes are required for signing up? Name
 Do you want to enable any of the following capabilities?
 What domain name prefix do you want to use? amplifysnsfedarationyyyyyyy-xxxxxxx
 Enter your redirect signin URI: http://localhost:8080/
? Do you want to add another redirect signin URI No
 Enter your redirect signout URI: http://localhost:8080/
? Do you want to add another redirect signout URI No
 Select the social providers you want to configure for your user pool:
Successfully added resource amplifysnsfedarationdff223d9 locally

Some next steps:
"amplify push" will build all your local backend resources and provision it in the cloud
"amplify publish" will build all your local backend and frontend resources (if you have hosting category added) and provision it in the cloud

hirosue@PC876 amplify-sns-fedaration % amplify push
✔ Successfully pulled backend environment prod from the cloud.
```

---

* ### **3. CongnitoのLINE連携（OpenID Connect）設定**

 ### **Step1. AWSにログインして、Cognitoのサービスページに移動します。**
 ### **Step2. ユーザプールを選択します。**

「ユーザープールの管理」を選択し、2.認証モジュールの追加で作成した「ユーザープール」を選択します。

対象の「ユーザープール」の名称は、Amplifyの状態（amplify status）で確認したリソース名（Resource name）で始まります。
 ### **Step3. 「ID プロバイダー」（OpenID Connect）を追加します。**

| 項目名 | 値  
|:------|:------
| プロバイダ名 | LINE（任意の値）     
| クライアントID | （1. LINE側（LINE Developers）の設定で設定した「チャンネル ID（Channel ID）」）     
| クライアントのシークレット（オプション| （1. LINE側（LINE Developers）の設定で「チャンネル シークレット（Channel secret）」     
| 属性のリクエストメソッド | GET（デフォルトのまま）     
| 承認スコープ    | profile mail openid     
| 発行者    | https://access.line.me       
| 識別子（オプション）    |   - 
| 認証エンドポイント    | https://access.line.me/oauth2/v2.1/authorize     
| トークンエンドポイント    | 	https://api.line.me/oauth2/v2.1/token     
| ユーザー情報エンドポイント    | https://api.line.me/v2/profile     
| Jwks uri    | https://api.line.me/oauth2/v2.1/verify
* （参考リンク）LINEとの連携設定

[LINE Developer 設定](https://blog.u-chan-chi.com/post/amplify-oidc-line-vue/#Line-Developer%E8%A8%AD%E5%AE%9A)
 ### **Step4. Step3で作成した「ID プロバイダー」（OpenID Connect）の属性マッピングを変更します。**

 2. 認証モジュールの追加で作成した「ユーザープール」はName属性が必須であるため、sub⇆Nameのマッピングを追加します。

 ### **Step5. アプリクライアントの設定でLINEログインを有効化します。**

作成されている2つのアプリクライアント（xxxxxxxxxxxxxx_app_clientおよびxxxxxxxxxxxxxx_app_clientWeb）に対し、LINE連携を有効化します（チェックして保存する）。

---

* ### **4. LINE側（LINE Developers）にリダイレクトURLの設定**

1. LINE側（LINE Developers）の設定で作成した「プロバイダー」にCognitoで作成された認証のリダイレクトURLを設定します。
### **Step1. Amplifyの設定ファイル（aws-exports.js）を確認して、Cognitoのドメインを確認します。**

2. 認証モジュールの追加の結果作成された、Amplifyの設定ファイル（aws-exports.js）のCognitoのドメインの箇所を確認します。

```php
% cat src/aws-exports.js | grep domain
        "domain": "amplifysnsfedarationyyyyyyy-xxxxxxx-prod.auth.ap-northeast-1.amazoncognito.com",
```
### **Step2. 1. LINE側（LINE Developers）の設定で作成した「プロバイダー」にCognitoで作成された認証のリダイレクトURLを設定します。**
「TOP」-「Cognito Test」-「Cognito Test」-「LINE Login」と移動して、リダイレクトURL（CallBack URL）を入力します。

設定する値はhttps://（Step1で確認したドメイン）/oauth2/idpresponseです。

* ### **フロント側の設定**
