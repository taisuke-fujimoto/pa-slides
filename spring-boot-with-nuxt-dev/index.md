---
marp: true
paginate: true
theme: gaia
footer: '© Phone Appli Inc.'
---
<!-- _class: lead -->

# Spring Boot + Nuxt.js 連携 (開発時)

---

## 概要

現在、フロントエンド FW 更新 PJ で使用している Nuxt.js と Spring Boot 連携について説明。

* Nuxt.js 説明 (サーバ開発者向け)

* 開発時の連携方法について
  * 本番時の連携方法は未確立 😢

---

## Nuxt.js とは

* Vue.js のフレームワーク

* Vue + ユーティリティ + ビルドツールのような感じ

* Spring Boot と同様に開発サーバ有り
デフォルト `localhost:3000`

* サーバサイドレンダリング (SSR) 可能
※ RTK では今のところ使用予定無し

---

## 連携時の課題

```
              +- localhost:3000 -+       +-- localhost:8080 --+
              |                  |       |                    |
browser ----> |   Nuxt Server    | ----> | Spring Boot Server |
              |                  |       |                    |
              +------------------+       +--------------------+
```

* Ajax

* CSRF トークン

* ログイン処理

---

## Ajax の課題

* フロント `localhost:3000` → サーバ `localhost:8080` の Ajax 通信は不可
  * Same-Origin Policy によりブラウザがブロック

↓

* Nuxt Proxy モジュールを使う
Nuxt サーバが `localhost:3000/api/xxx` への通信内容をそのまま `localhost:8080/api/xxx` に投げる

---

## CSRF トークン おさらい

* `GET` `HEAD` `TRACE` `OPTIONS` 以外で必要なトークン
(Spring Security の仕様)

* アクセスするブラウザでユニーク
ログイン成功時に変更される

---

## CSRF トークンの課題

* フロントで Spring の CSRF トークンが分からない

↓

* Spring Boot 設定
  * `CookieCsrfTokenRepository` を使う
  * cookie の httpOnly 属性を false に設定

cookie に CSRF トークンが入るので javascript で取得
**※ cookie はホストでしか判断しない (ポートが違っても OK)**

---

## ログイン処理の課題

* 特殊なログイン方法 (o365 / SAML / OIDC) ともにサーバ側 `localhost:8080` を経由する必要がある

↓

* ログイン実行時 `localhost:3000` → `localhost:8080` へブラウザの POST (not Ajax)
* ログイン処理後 `localhost:8080` → `localhost:3000` へリダイレクト
  * 認証 OK 時、認証済 セッション ID が cookie に入る
    以降の Ajax で使用可能

---

## 最後に : 本番時の連携方法の想定

* Nuxt サーバを実行しない設定で、フロントのビルド内容を Spring Boot の static リソースとして配置

* フロントとサーバが同じホスト・ポートになるので、Nuxt Proxy モジュールも動かさない

↓

ビルドツールとの戦いになりそう

---

## 補足

* Nuxt サーバのみで開発できる mock モード有り
  * API レスポンスを用意する必要があるが Spring Boot 無しで起動可能
