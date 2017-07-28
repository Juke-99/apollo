Apollo
======

[![Circle Status](https://circleci.com/gh/spotify/apollo.svg?style=shield&circle-token=5a9eb086ae3cec87e62fc8b6cdeb783cb318e3b9)](https://circleci.com/gh/spotify/apollo)
[![Codecov](https://img.shields.io/codecov/c/github/spotify/apollo.svg)](https://codecov.io/gh/spotify/apollo)
[![Maven Central](https://img.shields.io/maven-central/v/com.spotify/apollo-parent.svg)](https://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.spotify%22%20apollo*)
[![License](https://img.shields.io/github/license/spotify/apollo.svg)](LICENSE)

Apolloは私たちがマイクロサービスを書くときにSpotifyを使うJavaライブラリのセットです。ApolloはHTTPサーバとURIルーティングシステムのようなモジュールを含み、RESTful APIサービスを実装することにどこにでもあるHTTPサーバとURIルーティングシステムを作ります。

Apolloは長い間、Spotifyの製品で使われていました。バージョン1.0.0をリリースするために作業の一部として私たちはオープンにApolloの開発を移行しました。

Apolloに3つのメインライブラリがあります：

* [apollo-http-service](apollo-http-service)
* [apollo-api](apollo-api)
* [apollo-core](apollo-core)

もしメインAPIが十分に強力でない問題を解決する必要があるなら、[apollo-environment](apollo-environment)は、Apolloのコアな振る舞いをあなたに修正するために、より手を貸しましょう。

### Apollo HTTPサービス
[apollo-http-service](apollo-http-service)ライブラリはApolloモジュールの標準化されたアセンブリです。それはapollo-apiとapollo-coreの両方を組み込み、送受信によって、httpを使う標準apiサービスを取ってくるために他のモジュールと一緒につなぎます。

### Apollo API
[apollo-api](apollo-api)ライブラリはあなたが触れることが最もありそうなApolloライブラリです。それは自分のサービスルートとリクエスト/リプライハンドラを定義する必要があるツールを与えます。

ここで、例えば、私たちのサービスが文字列`"hello world"`によってパス`/`上でGETリクエストに応えるものを定義します：
```java
public static void init(Environment environment) {
  environment.routingEngine()
      .registerAutoRoute(Route.sync("GET", "/", requestContext -> "hello world"));
}
```

apollo-apiライブラリはあなたのリクエスト/リプライハンドラを定義するのを助けるための様々な方法を提供してくれます。あなたはレスポンスが（JSONのような）シリアライズをどのようにするかを特定することができます。[Apollo API Readme](apollo-api)でこのライブラリについてより詳しくわかります。

### Apollo Core
[apollo-core](apollo-core)ライブラリはあなたのサービスのライフサイクル（ローディング、スターティング、そしてストッピング）を管理します。あなたはapollo-coreを直接触れる必要があまりありません。"プレミング"（<- 原文ではplumbing）として単に思います。このライブラリについてより情報が欲しいのなら、[Apollo Core Readme](apollo-core)をみてください。

### Apollo Test
3つのメインApolloライブラリを一覧にすることも加えて、あなたのサービスのテストを書くことを助けるために私たちは[apollo-test](apollo-test)を呼ぶ追加のライブラリを持ちます。それはテスト中のサービスを準備し、送信リクエストレスポンスをモック（まね）するためのヘルパーを持ちます。

### Apollo入門
ApolloはMavenアーキテクチャのセットとして配布されています。Apolloはビルドツール（Maven、Ant + IvyもしくはGradle）関係なく入門することが簡単です。以下はとても単純ですが機能的なサービスです。より広範な例が[examples](examples)ディレクトリにあります。これらがリリースされるまで、あなたは`mvn install`を起動することでソースからApolloをビルドとインストールできます。

```java
public final class App {

    public static void main(String... args) throws LoadingException {
        HttpService.boot(App::init, "my-app", args);
    }

    static void init(Environment environment) {
        environment.routingEngine()
            .registerAutoRoute(Route.sync("GET", "/", rc -> "hello world"));
    }
 }
```

### Apolloメタデータ
エンドポイントのような、Apollo-basedサービスについて[Metadata](apollo-api-impl/src/main/java/com/spotify/apollo/meta/model)はランタイムを出します。Spotifyで私たちは私たちのランタイムサービスのトラックを保つのにこれを使いました。より詳細な情報は [ここ](https://apidays.nz/slides/iglesias_service_metadata.pdf)で理解することができます。

[spotify-api-example](examples/spotify-api-example)からの例:

`$ curl http://localhost:8080/_meta/0/endpoints`

```json
{
  "result": {
    "docstring": null,
    "endpoints":[
      {
        "docstring": "Get the latest albums on Spotify.\n\nUses the public Spotify API https://api.spotify.com to get 'new' albums.",
        "method": [
          "GET"
        ],
        "methodName": "/albums/new[GET]",
        "queryParameters":[],
        "uri": "/albums/new"
      },
      {
        "docstring": "Responds with a 'pong!' if the service is up.\n\nUseful endpoint for doing health checks.",
        "method": [
          "GET"
        ],
        "methodName": "/ping[GET]",
        "queryParameters": [],
        "uri": "/ping"
      },
      ...
    ]
  }
}
```

`$ curl http://localhost:8080/_meta/0/info`

```json
{
  "result": {
    "buildVersion": "spotify-api-example-service 1.3.1",
    "componentId": "spotify-api-example-service",
    "containerVersion": "apollo-http2.0.0-SNAPSHOT",
    "serviceUptime": 778.249,
    "systemVersion": "java 1.8.0_111"
  }
}
```

### リンク

[Introduction Website](https://spotify.github.io/apollo)<br />
[JavaDocs](https://spotify.github.io/apollo/maven/apidocs)<br />
[Maven site](https://spotify.github.io/apollo/maven)

### 図解

[![Apollo set-up](https://cdn.rawgit.com/spotify/apollo/master/website/source/set-up.svg)](website/source/set-up.svg)

[![Apollo in runtime](https://cdn.rawgit.com/spotify/apollo/master/website/source/runtime.svg)](website/source/runtime.svg)

## 行動規範
このプロジェクトは[行動規範を開く][code-of-conduct]を支持します。参加することで、あなたはこのコードに貢献できると思います。

[code-of-conduct]: https://github.com/spotify/code-of-conduct/blob/master/code-of-conduct.md
