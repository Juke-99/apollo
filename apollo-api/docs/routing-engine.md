# Routing Engine

[`RoutingEngine`](/apollo-api/src/main/java/com/spotify/apollo/Environment.java#L72)は[`Environment`](/apollo-api/src/main/java/com/spotify/apollo/Environment.java)の一部で、[`Route`](/apollo-api/src/main/java/com/spotify/apollo/route/Route.java)もしくは
[`RouteProvider`](/apollo-api/src/main/java/com/spotify/apollo/route/RouteProvider.java)インスタンスを登録するために使われます。

## 例

```java
void init(Environment environment) {
  environment.routingEngine()
      .registerAutoRoute(Route.sync("GET", "/ping", requestContext -> "pong"))
      .registerAutoRoutes(new MyResource());
}
```

それ自身のクラスとしての`MyResource`の実装。

```java
class MyResource implements RouteProvider {

  @Override
  public Stream<? extends Route<? extends AsyncHandler<?>>> routes() {
    return Stream.of(
        Route.sync("GET", "/v1/address/<name>", requestContext -> "!"/* do work */),
        Route.sync("PUT", "/v1/address/<name>", requestContext -> "!"/* do work */)
    );
  }
}
```

## 重複ルートパス

他のより一般的な重複ルートの特定ケースの一つを作る方法で重複する二つのルートを定義したとします。このような重複の例は：

1. `/foo/bar`
2. `/foo/<arg>`

したがってルート2の特定ケースであるルート1を作ることで、ルート2は`/foo/bar`の呼び出しとマッチすることができます。これのおかげで、`RoutingEngine`はより一般的なルートよりも前により特定のルートを経由します。

## パスパラメータ

スラッシュを含めて、パスパラメータをマッチするために`/foo/<arg:path>`を使います。
