# Routes

同期と非同期ごとに、リクエストハンドラの型を基本的に基にする2つのルートの型があります。それぞれのルートはメソッド、URI、文章文字列、ハンドラのような基本情報を定義します。

ルートを作るための2つのメインメソッド：

* `Route.sync(String method, String uri, SyncHandler<T> handler)`
* `Route.async(String method, String uri, AsyncHandler<T> handler)`


`Route<AsyncHandler<T>>`型のルートを予想する`RoutingEngine`から、`Route.sync`と`Route.async`両方は`Route<AsyncHandler<T>>`を返し、そのおかげでルートを登録するのに直接使うことができます。

ルートは`RoutingEngine.registerAutoRoute(s)()`を加えます。でないと`Middlewares::autoSerialize`もしくは`Middlewares::apolloDefaults`は`AutoSerializer`と一緒にそのレスポンスペイロードをシリアライズします。ルートは`RoutingEngine.registerRoutes()`が`Response<ByteString>`を返すことで加わります。そうするとさらに処理が加わるわけではありません。

## ルートハンドラリプライタイプ

明白な`T`型を返す代わりに、ルートハンドラは[`Response<T>`](/apollo-api/src/main/java/com/spotify/apollo/Response.java)を返します。あなたがリプライ（[Response](/apollo-api/docs/response.md)をみてください）についての追加情報をちゃんとわかるラッパーです。

同期/非同期とプレイン/レスポンスの組み合わせのマトリックスはこのように思われています：

|      型        | `SyncHandler<T>` | `AsyncHandler<T>` |
|:---------------: | -------------- | --------------- |
|     **`T`**      | `T` - 同期プレインペイロードは状態コード`200 OK`をリプライします | `CompletionStage<T>` - 非同期プレインペイロードは状態コード`200 OK`をリプライします |
| **`Response<T>`** | `Response<T>` - カスタム状態コードとハンドラによる同期ペイロードは | `CompletionStage<Response<T>>` - カスタム状態コードとハンドラによる非同期ペイロード |

## Route providers

[`RouteProvider`](/apollo-api/src/main/java/com/spotify/apollo/route/RouteProvider.java)はルートの[`Stream`](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)を作るための関数型インターフェースです。

これは、アプリケーションがいくつかの論理的な方法でルートをグループ化し、すべてを一括して扱う必要がある場合に便利です。リソースをグループ化したエンドポイントによるRESTful APIは普通のユースケースです。

```java
static class BlogPost implements RouteProvider {

  @Override
  public Stream<? extends Route<? extends AsyncHandler<?>>> routes() {
    return Stream.of(
        Route.sync("GET", "/blogpost/<id>", ctx -> getPost(ctx.pathArgs().get("id"))),
        Route.sync("POST", "/blogpost", ctx -> createPost(ctx))
    );
  }

  // Post getPost(String id) { ... }
  // Post createPost(RequestContext context) { ... }
}

public static void init(Environment environment) {
  // ...
  environment.routingEngine()
      .registerAutoRoutes(new BlogPost());
}
```


複数ルートをより有用にするためには、[`apollo-extra`](/apollo-extra)モジュールをみてください。
