---
title: "Node.jsでHTTP/2を動かしてみる"
date: 2017-12-09T04:22:57+09:00
tags: ["Node.js", "HTTP/2", "Javascript"]
categories: ["ビトル"]
---
どうも、ビトルです。

今回は[Node.js Advent Calendar 2017](https://qiita.com/advent-calendar/2017/nodejs)の9日目の記事になります。

<br>

この記事ではNode.jsで現在コア機能に実験的に導入されているHTTP/2について書きたいと思います。

<br>

### Serverside

<br>

単純なHTTPサーバとしての処理はこんな感じに実装できます。

<br>

```js
const http2 = require('http2');
const fs = require('fs');

const server = http2.createSecureServer({
  key: fs.readFileSync('localhost-privkey.pem'),
  cert: fs.readFileSync('localhost-cert.pem')
});
server.on('error', (err) => console.error(err));
server.on('socketError', (err) => console.error(err));

server.on('stream', (stream, headers) => {
  stream.respond({
    'content-type': 'text/html',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(8080);

```
ChromeやFirefoxなどのブラウザはHTTP/2を利用する際はSSLが必須となっているので、証明書を用意する必要があります。
証明書は自己証明書でも作成できます。

<br>

```sh
openssl req -x509 -newkey rsa:2048 -nodes -sha256 -subj '/CN=localhost' \
  -keyout localhost-privkey.pem -out localhost-cert.pem
```


### Clientside

<br>

もしHTTP/2に対応しているサイトにリクエストを送りたい場合はこんな感じになります。

<br>

```js
const http2 = require('http2');
const client = http2.connect('https://twitter.com');
client.on('socketError', (err) => console.error(err));
client.on('error', (err) => console.error(err));

const req = client.request({ ':method': 'GET', ':path': '/' });

req.on('response', (headers, flags) => {
  for (const name in headers) {
    console.log(`${name}: ${headers[name]}`);
  }
});

req.setEncoding('utf8');
let data = '';
req.on('data', (chunk) => { data += chunk; });
req.on('end', () => {
  console.log(`\n${data}`);
  client.destroy();
});
req.end();
```

### Server Push

<br>

HTTP/2の最大の特徴は何と言ってもサーバープッシュですね。
ざっくり言うとHTTP/2ではデータのやり取りが非同期的にできるので、jsやcssなどのコンテンツを優先的に事前に読み込んでページのレンダリング速度を向上させることができます。

<br>

```js
const http2 = require('http2');
const fs = require('fs');
const mime = require('mime');

const { HTTP2_HEADER_PATH } = http2.constants

function getFile (path) {
  const filePath = `${__dirname}/public${path}`;
  try {
    const content = fs.openSync(filePath, 'r');
    const contentType = mime.getType(filePath);
    return {
      content,
      headers: {
        'content-type': contentType
      }
    };
  } catch (e) {
    return null;
  }
}

// ファイルをストリームにプッシュする
function push (stream, filePath) {
  const file = getFile(filePath);
  if (!file) {
    return;
  }

  const pushHeaders = { [HTTP2_HEADER_PATH]: filePath };
  stream.pushStream(pushHeaders, (pushStream) => {
    pushStream.respondWithFD(file.content, file.headers);
  })
}

// リクエストハンドラ
function onRequest (req, res) {
  if (reqPath === '/index.html') {
    push(res.stream, '/my-js-file.js');
    push(res.stream, '/my-css-file.css');
  }

  // ファイルを送る
  res.stream.respondWithFD(file.fileDescriptor, file.headers);
}

const server = http2.createSecureServer({
  key: fs.readFileSync('localhost-privkey.pem'),
  cert: fs.readFileSync('localhost-cert.pem')
}, onRequest);

server.on('error', (err) => console.error(err));
server.on('socketError', (err) => console.error(err));

server.listen(8080, (err) => {
  if (err) {
    console.error(err);
    return
  }
});
```

↑の実装では、index.htmlにアクセスがあった時に`my-js-file.js`と`my-css-file.css`をブラウザからのコンテンツに対するリクエストが来る前に優先的に読み込ませるようにしています。

<br>

## まとめ
いかがでしょうか。HTTP/2の普及率はまだ発展途上ですが、多数のjsやcssファイルを読み込む必要があるページは多く
またスマートフォンでの閲覧を考えると今後はHTTP/2を利用したページは当たり前になってくるのではないかと思います。
ページの読み込み速度は常に出てくる課題なので、Node.jsのコア機能に是非追加されることを希望しています！！
<br>
それではまた！！

<br>
