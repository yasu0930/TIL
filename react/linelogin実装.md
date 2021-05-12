# **ReactとLaravelを使ったLINEログインの実装**
* ### **リダイレクトページを作成**
前提としてユーザーが「ログイン」ボタンを押した時に遷移するページを作成しておく。

APIのapi/loginからリダイレクトURLを取得する

</>redireact.js
```php
import React, { useState, useEffect } from "react";
import axios from 'axios';

const Redirect = () => {
    const API_URL = process.env.REACT_APP_API_URL;

    // RedirectURLの取得
    useEffect(() => {
        axios.get(API_URL + "api/login").then(res => {
            window.location.href = res.data.redirect_url;
        }).catch(err => {
    
        })
    }, []);

    return (
        <div>
            <h1>リダイレクトしています</h1>
        </div>
    );
}

export default Redirect;
```
useEffectでリダイレクトURLをsetXX系で管理していないのは、値を更新しても、即座には変更さないため。その値を後続の処理でも使いたい場合は、変数に入れて使いまわしたりしてください。

---

* ### **リダイレクトURLを取得するAPIを作成**
reactで呼び出したapi/loginをLaravelで実装します。

リダイレクトURLはLaravel Socialiteを使って取得します。
```php
use Laravel\Socialite\Contracts\Factory as Socialite;

    // SocialiteからリダイレクトURLを取得
    public function redirect()
    {
        return response()->json(['redirect_url' => $this->socialite->driver('line')
                                ->redirect()->getTargetUrl()]);
    }
```
注意点としてSocialiteではセッション（Session）を利用します。

Kernel.phpのmiddlewareGroupsに新たなsessionグループを追加する。

</>App\Http\Kernel.php
```php
protected $middlewareGroups = [
    'session' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
    ],
];
```
Controllerのミドルウェアに「session」を指定すれば、セッションを利用できるようになります。

これでLaravel側の設定は完了です。


---

* ### **LNE認証後のCallbackページを実装**
ユーザーが先ほど取得したリダイレクトURLに遷移してアプリへ戻ってきました。

戻ってきたURLは以下のようになります。

「https:/line-login-app.me/callback?code=xxx&state=xxx」

戻ってきたURLのクエリをwindow.location.searchで取得し、APIで送信してLaravel側で処理してもらう。

</>Callback.js
```php
import React, { useState, useEffect } from "react";
import axios from 'axios';

const CallBack = () => {
    const [token, setToken] = useState("");
    const API_URL = process.env.REACT_APP_API_URL;

    useEffect(() => {
        let query = window.location.search;

        axios.get(API_URL + "api/callback" + query).then(res => {
            setToken(res.deta.Access-Token);
        }).catch(err => {
    
        })
    }, []);

    return(
        <h1>LINEログインしています。</h1>
    );
}

export default CallBack;
```

---

* ### **Laravel側でAccessTokenを返す**
```php
public function callback()
{
    try {
        $user = $this->socialite->driver('line')->stateless()->user();
    } catch (Exception $e) {
        return response()->json(['error' => 'Invalid credentials provided.'],422);
    }

    $authUser = User::firstOrCreate(
        [
            'line_id' => $user->line_id
        ]
    );

    $token = $authUser->createToken('token-name')->plainTextToken;

    return response()->json($token, 200, ['Access-Token' => $token]);
}
```

---

* ### **返ってきたAccessTokenをuseStateに保存する**
もう一度react側のコールバック処理を見てみよう。
```php
let query = window.location.search;

axios.get(API_URL + "api/callback" + query).then(res => {
    setToken(res.deta.Access_token);
}).catch(err => {.....
```
取得したAccessTokenとuseStateへ保存しています。

これでReactとLaravelを使ったLINEログイン機能の実装完了です。
