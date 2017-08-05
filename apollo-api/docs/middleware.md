# ミドルウェア

> このセクションはルートハンドラについていくつか話しているので、[ルート](/apollo-api/docs/routes.md)ドキュメントのはじめを呼んでください。
>
> Java 8ラムダの大きな使用にもなります。ラムダにはもう少し詳しくこの設定でオブジェクト指向プログラミングとどのように関係するか理解するために、[Classes to Lambdas README](/apollo-api/docs/class-to-lambda.md)を呼んでください。

ミドルウェアはルートハンドラの振る舞いを授かるのに使うことができる関数です。これはコード重複を避けるためのいくつかのルートの普通の機能性を加えるのに使うことができます。

なぜなら _inner_ ハンドラを授かる関数としてちょうどよく、普通の機能性のたくさんのクラスを実装するための多様なツールを提供してくれるからです。幾らかの使用例と一緒にMiddlewareを実装することができる幾らかの基本的なパターンがあります。

ミドルウェアは以下のことができます：

* 何かをして、内側のハンドラを透過的に呼び出すことで続行
 * ロギングリクエスト
 * 統計収集
 * 他のコードパスをダークロード
* 内部ハンドラを呼び出さず、代わりにリクエストに直接応答することを決定
 * ACLテェック
 * リクエスト検証
* 内側のハンドラを呼んだ後に、リプライ前にレスポンスを編集
 * キャッシングハンドラの追加
 * リクエストハンドラを基にしたペイロードのレスポンス表現の選択
* 幾らかの方法でリクエストコンテキストを編集した後に内側のハンドラの呼び出し
 * コード移植間の互換性上の理由のための受信リクエストの編集
 * 詳しく調べられたリクエストクライアントの振る舞いの編集
* 内側のハンドラを複数回の呼び出し
 * 取り戻し可能な障害の状況のリトライ

## 一般例

これはミドルウェアがどのように実装されるかです。コメントは、上記のさまざまなパターンがどのように実装できるか、それらが`requestContext`と` innerHandler`とどのように互いに干渉し合うかを説明します。

```java
static <T> SyncHandler<Response<T>> myMiddleware(SyncHandler<T> innerHandler) {
  return requestContext -> {
    // Check condition before continuing dispatch
    final boolean isOkToProceed = condition(requestContext);

    if (!isOkToProceed) {
      return Response.forStatus(Status.UNAUTHORIZED);
    }

    LOGGER.info("Handling request to {}", requestContext.request().uri());

    // Decorate RequestContext
    final RequestContext loggingOutgoingCallsContext = loggingContext(requestContext);

    // Call inner handler
    final T innerResponse = innerHandler.invoke(loggingOutgoingCallsContext);

    return Response.forPayload(innerResponse)
        .withHeader("X-Added-Header", "With Some Value");
  };
}
```

このミドルウェアはルート定義と一緒に使うことができます。

```java
static void init(Environment environment) {
  environment.routingEngine()
      .registerAutoRoute(
          Route.<SyncHandler<String>>create("GET", "/foo", requestContext -> "hello world")
              .withMiddleware(Small::myMiddleware)
              .withMiddleware(Middleware::syncToAsync))
  ;
}
```

_私たちは自分のミドルウェアが動くために定義したハンドラタイプである`SyncHandler<T>`を取得するために`Route.<SyncHandler<String>>create()`を使う必要があることに注意してください。最後に、私たちはフレームワークが呼ぶことができる`AsyncHandler<T>`の中のルートハンドラを回る`Middleware::syncToAsync`ミドルウェア（apollo-apiで定義されています）を採用します。_

## 発展： カスタムコンテキストタイプを作る

アプリケーションで、私たちはリクエストをページングし、証明するためのマシンを持ちます。私たちはこれらのコンテキストタイプを手本にします：

```java
interface AuthContext {
  Optional<String> user();
}

interface PagingContext {
  int page();
}
```

証明され、ページされたリクエストの情報を構成する二つの単純なインターフェースです。二つのコンテキストがお互いにそれほど関係しないことに注意してください。私たちはどのように一緒に使用されるのかを見ていきます。

これらのコンテキストをどのように作成するかの実装は単なる関数です：

```java
PagingContext page(RequestContext c) {
  int page = c.request().getParameter("page")
      .map(Integer::parseInt)
      .orElse(0);

  return () -> page;
}

AuthContext auth(RequestContext c) {
  String userName = getUsername(c);

  return () -> Optional.ofNullable(userName);
}
```

ここまでは順調ですね。ここからは一歩進んで行きます。まさに明確なJavaとApolloはまだ含んでいません。

次に、私たちは明確な`String`レスポンスを作るためのコンテキストを両方使うエンドポイントを回します。

```java
String whereAmI(AuthContext authContext, PagingContext pagingContext) {
  return authContext.user() + ", you're on page " + pagingContext.page();
}
```

では、このエンドポイントを`AuthContext`と`PagingContext`の両方を持つ`Route`にどのようにバインドするのでしょうか？なるべく、私たちは次のように単純なものを好みます：

```java
Route.create("GET", "/test", authContext -> pageContext -> whereAmI(authContext, pageContext));
```

これはほとんど動作します。`Route.create(...)`はそのハンドラとして何かの`T`型を実際は持ってきます。しかし、Apolloは`AsyncHandler<T>`の実装出ない限りハンドラでするべきことを知りません。この場合（型を少し助けること）、ハンドラは`Function<AuthContext, Function<PagingContext, String>>`です。私たちは`AsyncHandler<String>`の中でどのように変わるかApolloと対話する必要がただあります。

このため、私たちは二つの関数型インターフェースをはじめに宣言します：

```java
interface Authenticated<T> extends Function<AuthContext, T> {}
interface Paged<T> extends Function<PagingContext, T> {}
```

`Function<AuthContext, Function<PagingContext, String>>`、すなわち`Authenticated<Paged<T>>`のより良い名前を与えることが可能です。

私たちは自分の二つのハンドラタイプに置き換わる`Middleware`を作ります。それは`auth(RequestContext)`と`page(RequestContext)`の両方を使います。

```java
<T> Middleware<Authenticated<Paged<T>>, AsyncHandler<T>> authPaged() {
  return ap -> (requestContext) -> {
    T payload = ap
        .apply(auth(requestContext))
        .apply(page(requestContext));
    return immediateFuture(forPayload(payload));
  };
}
```

アプリケーションが必要とするあらゆるコンテキストの組み合わせに対して、またはすべてのコンテキストをまとめて含むより大きなタイプのコンテキストのいずれかを作成できます。重要なことはそれがたくさんの`Route`を定義することで再利用されるということです。

最後に、私たちはハンドララムダの型を推測する手助けをするファクトリーメソッドを使うのに`Route`を定義し、同じくらい素晴らしく全体を見せます：

```java
Route<AsyncHandler<String>> route =
    Route.with(authPaged(), "GET", "/test", auth -> page -> whereAmI(auth, page));
```

`RoutingEngine`によって開始し、幾らかのリクエストを作ります。以下は実行した例です：

```
$ curl 'http://localhost:8080/ping/test'
Optional.empty, you're on page 0

$ curl 'http://rouz@localhost:8080/ping/test?page=4'
Optional[rouz], you're on page 4
```
