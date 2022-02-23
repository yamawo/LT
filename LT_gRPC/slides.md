---
theme: apple-basic
# OGPやタブ名などで使う
title: "gRPCについて学びました"
# 使用される画像群
background: https://source.unsplash.com/collection/94734566/1920x1080
# ライト・ダークモードの選択
colorSchema: "light"
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
layout: intro
# persist drawings in exports and build
drawings:
  persist: false
# fonts:
#   sans: "Noto Sans JP"
---

# gRPC 学びました

<div class="absolute bottom-10">
  <span class="font-700">
    山﨑 翔太
  </span>
</div>

---

# gRPC とは

RPC を HTTP/2 で実現した通信規格のこと。

## RPC(Remote Procedure Call/遠隔手続き呼び出し)

👉 ネットワーク上の他端末と通信するための仕組みのこと。スタブと呼ばれるコードを通して、アプリケーションが他端末上のサーバーアプリケーションのメソッドを直接呼び出せる。

![RPC図](/RPC図.png)

これをより使いやすく高性能にしたもの

<!-- <style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style> -->

---

<img src="/gRPC.png" class="my-0 mx-auto" />

---

# 何が出来る・出来ないのか

## 👍

- Protocol Buffers というシリアライズフォーマットを使ってデータをシリアライズし、高速な通信を実現
- Protocol Buffers を用いることで言語に依存せず、一つの .proto ファイルで 12 言語の実装を生成できる
- microservices 間の通信で高い親和性 ◎
- HTTP/2 を基に作られており、簡単に HTTP/2 の恩恵を受けられる
- SSL 通信がデフォルト
- 4 つの API タイプを作成でき、様々な種類/用途の API を開発できる

1. Unary (一般的なやつ：1 req, 1 res) 👉 モバイルクライアントや Web クライアントをバックエンドと繋げる際に
2. Server Streaming (client:1 req, server:複数 res) 👉 push 通知
3. Client Streaming (client: 複数 req, server: 1 res) 👉 ライブストリーミングなど、大容量の動画データを分割してリクエストする際に。
4. Bi Directional Streaming (client:複数 req, server:複数 res) 👉 チャット、ゲーム

---

## 👎

- ブラウザ(gRPC-Web)から直接 gRPC サーバーを叩くことは現状できない(envoy や nginx みたいなプロキシを挟む必要あり)

  > HTTP/2 gRPC spec3 をブラウザに実装することは現在不可能です。なぜなら、リクエストに対して十分にきめ細かい制御が可能なブラウザ API が存在しないからです。例えば、HTTP/2 を強制的に使用する方法はありませんし、たとえあったとしても、生の HTTP/2 フレームはブラウザでアクセスすることができません。 [（gRPC 公式より）](https://grpc.io/blog/state-of-grpc-web/#the-grpc-web-spec)

- データがシリアライズされるのでデバッグしにくい（未経験）
- 公式含めて日本語のドキュメントが少ない

---

<div class="my-0 mx-auto">
  <h1 class="text-center"> さっきから Protocol Buffers ってなんだ？</h1> 
</div>

---

# Protocol Buffers

データや通信方式を定義するインターフェイス言語（他には GraphQL や OpenAPI）

```ts
syntax = "proto3";

package chat;

service Chat {
  rpc GetMessages (google.protobuf.Empty) returns (stream Message);
  rpc PostMessage (Message) returns (Result);
}

message Message {
  string name = 1;
  string message = 2;
  google.protobuf.Timestamp createdAt = 3;
}

message Result {
  bool result = 1;
}
```

---

## 特徴

- コンパイルすると、各言語で getName() や getMessage()、GetMessages()、PostMessages() などなどの処理が実装されたファイルが出来上がる
- 「レスポンスとリクエスト」のデータ形式でやりとりが出来る 👉 RESTful API だとエンドポイントを定義してそこにリクエストを叩かないといけない
- ストリーミングを使うか 👉 stream と付けるだけ
- エンドポイントのドキュメントがなくても .proto ファイルを見れば理解出来る

---

# 開発の流れ

<div class="text-center">
proto ファイルを作成して<br /><br />  
👇<br /><br />
利用するサーバーそれぞれでコンパイル<br /><br />  
👇<br /><br />
作成されたデータを受け渡すメソッドを用いてロジックを実装していく<br /><br /> 
👇<br /><br />
のみ<uim-rocket class="text-3xl text-red-400 mx-2" />

</div>

---

# 最後に

私は新卒研修最後の成果物作成時に、
<br />
PHP の gRPC サーバーを使ったアプリケーションを作ろうと意気込んでいたのですが、
研修当時は gRPC 公式が PHP はサポートしないと明言をしていて、チュートリアルとか proto ファイルのコンパイラとかありませんでした。（なんか有志の？コンパイラがあった）  
苦しみつつ調べながら苦労して実装の基盤を作ったのに最終的にコンパイラが上手く動かず、半分以上の時間をドブに捨てた過去があります。  
<br />
どうやら gRPC の特徴である streaming の機能と PHP の仕様が上手く噛み合わなくて web-socket で良いでしょってなっていたみたいです。（調べたらいくつか記事が出てきます！）
<br /><br />
それが最近ついに公式で PHP での開発をサポートし始めたらしくて、やっとか！！ということで Go を使い触ってみました。（PHP でやれ）

---

<img src="/owari.jpeg" class="my-0 mx-auto" />
