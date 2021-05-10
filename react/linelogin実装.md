# **ReactとLaravelでlineloginの実装**
* ### **リダイレクトページを作成**
前提としてユーザーが「ログイン」ボタンを押した時に遷移するページを作成しておく。

APIのapi/loginからリダイレクトURLを取得する
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
    public function redirectToProvider()
    {
        return response()->json(['target_url' => $this->socialite->driver('line')
                                ->redirect()->getTargetUrl()]);
    }
```