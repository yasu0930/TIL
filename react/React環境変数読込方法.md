# **Reactの環境変数をdotenv-cliで読み込む**
* ### **dotenv-cliを導入**
dotenv-cliをインストール
```php
npm install --save-dev dotenv-cli
```

* ### **環境変数のファイルを作成**
```php
├ .env                # ローカル環境 *今回はすでに作成済みでローカルでの作業
├ .env.development    # 開発環境
├ .env.staging        # 検証環境
├ .env.production     # 本番環境
```

</>.env
```php
REACT_APP_API_URL=https:-----
```
※Reactで環境変数を使用する場合、環境変数名の先頭にREACT_APP_を付ける必要がある

* ### **Reactで環境変数を使う**
先ほど定義した環境変数をprocess.envから取り出します。

</>index.js
```php
const API_URL = process.env.REACT_APP_API_URL;
console.log(API_URL);
```

* ### **サーバーの再起動**
環境変数についてのコードの変更を行った際は、変更を反映するためにサーバーを再起動する必要があるため。
