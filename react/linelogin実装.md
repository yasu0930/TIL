# **Reactでlineloginの実装**
* ### **リダイレクトページを作成**
前提としてユーザーが「ログイン」ボタンを押した時に遷移するページを作成しておく。

APIのapi/loginからリダイレクトURLを取得する
```php
import React, { useState, useEffect } from "react";
import axios from 'axios';

const Redirect = () => {
    const API_URL = process.env.REACT_APP_API_URL;

    // ブラウザリロード時にRedirectURLの取得
    useEffect(() => {
        const getAuthUrl = async () => {
            const res = await axios.get(API_URL + "api/login");
            window.location.href = res.data.redirect_url;
        }
        getAuthUrl()
    }, []);

    return (
        <div>
            <h1>リダイレクトしています</h1>
        </div>
    );
}

export default Redirect;

```
useEffectでリダイレクトURLをsetXX系で管理していないのは、値を更新しても、即座には変更されないため。その値を後続の処理でも使いたい場合は、変数に入れて使いまわしたりしてください。