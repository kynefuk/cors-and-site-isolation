# Site Isolation の動作確認

## 起動

```
docker-compose up -d
```

---

## CORB(`Cross Origin Read Blocking`)

`<iframe>`以外の HTML タグを使った、Cross Origin なリソースの Origin のレンダラプロセスへの展開を防止するためのブラウザのセキュリティ機構。

`<img`・`<script>`・`<style>`・`<font>`等を使用したリソースのロードについて、レスポンスが以下の`Content-Type`だった場合は、ブラウザはレスポンスを破棄するため、レンダラプロセスのメモリ上に展開されない。

- HTML(text/html)
- XML(text/xml・application/xml)
- JSON(text/json・application/json)

ブラウザで http://localhost:10000/corb/corb.html にアクセスするとコンソールログにエラーが表示される。

```
Cross-Origin Read Blocking (CORB) blocked cross-origin response http://localhost:20000/corb/loaded.html with MIME type text/html. See https://www.chromestatus.com/feature/5629709824032768 for more details.
```

---

## CORP(`Cross-Origin-Resource-Policy`)

CORB の対象外となるリソース(`.jpg`等の画像ファイルや`.js`等)に対して、CORB と同等の保護を行うためのヘッダ。
以下のように CORP のヘッダをリソースに付与すると、そのリソースがどの Origin から読まれて良いかを設定できる。
例えば、`same-site`や`same-origin`を指定することで、自分のサーバでホストしているリソースをクロスオリジンな罠サイトなどでロードされることを防止できる。

- `same-site`
  - Schemelessly Cross-Site からのリソース要求に対して、レンダラプロセスへのリソースのロードブロックされるようになる
- `same-origin`
  - Cross-Origin からのリソース要求に対しては、レンダラプロセスへのリソースのロードがブロックされるようになる
- `cross-origin`
  - `Cross-Origin-Embedder-Policy`ヘッダのための値なので、指定されてもレンダラプロセスへのリソースのロードがブロックされることはない

ブラウザで http://localhost:10000/corp/index.html にアクセスするとコンソールログに以下のエラーと共に`issue`が表示される。

```
# コンソールのエラー
GET http://localhost:20000/corp/loaded.js net::ERR_BLOCKED_BY_RESPONSE
```

```
# コンソールのissue
Specify a more permissive Cross-Origin Resource Policy to prevent a resource from being blocked

Your site tries to access an external resource that only allows same-origin usage. This behavior prevents a document from loading any non-same-origin resources which don’t explicitly grant permission to be loaded.
To solve this, add the following to the resource’s HTML response header:
Cross-Origin-Resource-Policy: same-site if the resource and your site are served from the same site.
Cross-Origin-Resource-Policy: cross-origin if the resource is served from another location than your website. ⚠️If you set this header, any website can embed this resource.
1 request
Request	Parent Frame	Blocked Resource
loaded.js
http://localhost:10000/corp/index.html
```

---

## COEP(`Cross-Origin-Embedder-Policy`)

HTML 内で読み込んでいる全てのサブリソースに CORP が設定されていることを強制するヘッダ。
CORP が設定されていないサブリソースの読み込みはブラウザによってブロックされる。

ブラウザで http://localhost:10000/coep/index.html にアクセスするとコンソールログに以下のエラーと共に`issue`が表示される。

```
# コンソールのエラー
GET http://localhost:20000/coep/loaded.js net::ERR_BLOCKED_BY_RESPONSE
```

```
# コンソールのissue
Specify a Cross-Origin Resource Policy to prevent a resource from being blocked

Your site tries to access an external resource that only allows same-origin usage. This behavior prevents a document from loading any non-same-origin resources which don’t explicitly grant permission to be loaded.
To solve this, add the following to the resource’s HTML response header:
Cross-Origin-Resource-Policy: same-site if the resource and your site are served from the same site.
Cross-Origin-Resource-Policy: cross-origin if the resource is served from another location than your website. ⚠️If you set this header, any website can embed this resource.
1 request
Request	Parent Frame	Blocked Resource
loaded.js
http://localhost:10000/coep/index.html
```

---

## COOP(`Cross-Origin-Opener-Policy`)

#TODO
