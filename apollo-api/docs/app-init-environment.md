# AppInit & Environment

Apolloアプリケーションは直接実装するかメソッドリファレンスを通してかのいずれかの[`AppInit`](/apollo-api/src/main/java/com/spotify/apollo/AppInit.java)インターフェースを初期化させます。これはサービスを起動するのとテストするための異なる初期メソッドを持つことができます。

```java
public interface AppInit {
  void create(Environment environment);
}
```

普通のアプリケーションは[`Environment.config()`](/apollo-api/src/main/java/com/spotify/apollo/Environment.java#L58)から値を読み込み、何かしらの特定のアプリケーションリソースを準備します。シャッタダウンするときにちゃんと閉じるために、これらのリソースは[`Environment.closer()`](/apollo-api/src/main/java/com/spotify/apollo/Environment.java#L72)を登録させるべきです。リソースを加え、アプリケーションは[`Environment.routingEngine()`](/apollo-api/src/main/java/com/spotify/apollo/Environment.java#L65)を使う1つ以上のエンドポイントを登録するべきです。

```java
public class DataService {

  /**
   * An implementation of the AppInit functional interface which simply sets up a
   * connection with a cassandra cluster based on some configuration values
   * and registers an address resource that will interact with it.
   */
  static void init(Environment environment) {
    String clusterName = environment.config().getString("data_store/cluster_name");
    String keySpace = environment.config().getString("data_store/key_space");
    String tableName = environment.config().getString("data_store/table");

    ClusterConnection cassandraClusterConnection = createClusterConnection(
        environment.domain(), clusterName, keySpace, tableName);

    environment.closer().register(cassandraClusterConnection);

    RouteProvider addressResource = new AddressResource(cassandraClusterConnection);
    environment.routingEngine().registerAutoRoutes(addressResource);
  }

  private static class AddressResource implements RouteProvider {

    private final String clusterConnection;

    AddressResource(ClusterConnection cassandraClusterConnection) {
      this.clusterConnection = cassandraClusterConnection;
    }

    @Override
    public Stream<? extends Route<? extends AsyncHandler<?>>> routes() {
      return Stream.of(
          Route.sync("GET", "/v1/address/<name>", requestContext ->
              getAddress(requestContext.pathArgs().get("name"))),

          Route.async("PUT", "/v1/address/<name>", requestContext ->
              putAddress(requestContext.pathArgs().get("name"),
                         requestContext.request().payload()))
      );
    }

    // implementations of getAddress and putAddress
  }

  /**
   * The main entry point for the service, referencing init
   */
  public static void main(String... args) throws LoadingException {
    HttpService.boot(DataService::init, "ping", args);
  }
}
```
