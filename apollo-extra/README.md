# Apollo Extra

このApolloライブラリはゆとりを持たせる幾らかのユーティリティを含みます。

## com.spotify.apollo.concurrent

`ListenableFuture`と`CompletionStage`の間で動くことを簡単にするいくつかのユーティリティを定義します。使用例：

```java
    ListenableFuture<Message> future = listenableFutureClient.send(myRequest);

    CompletionStage<Response<String>> response = Util.asStage(future)
        .thenApply(message -> Response.forPayload(message.data()));
```

また、[`ExecutorServiceCloser`](src/main/java/com/spotify/apollo/concurrent/ExecutorServiceCloser.java)のユーティリティを定義します。ライフサイクル管理のApollo `Closer`によるアプリケーション特有の`ExecutorService`インスタンスを登録することが便利になります。

## com.spotify.apollo.route

幾らかのシリアライズするミドルウェアとバージョニングエンドポイントのユーティリティを含みます。

## com.spotify.apollo.logging

注意: リクエスト処理デコレーターを利用してから、これらのユーティリティは廃止予定で、実際のレスポンスと何かをログする間の不一致を導くことで、リクエストもしくはレスポンスを変更するかもしれません。ロギングを準備するための好ましい方法の記述については[HttpService README](../apollo-http-service/README.md)をみてください。

ロギングユーティリティを含む、より一般的なものとして、Apache HTTPD'combined'フォーマットを使うログのデフォルト実装によって、リクエスト送信の通知を送ることができる解決策です。

アクセスログファイルに次のものを送るために、同様の構成を使います：

```
    <appender name="ACCESSLOG" class="ch.qos.logback.core.FileAppender">
        <file>/path/to/access.log</file>
        <encoder>
            <pattern>%msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.spotify.apollo.logging.RequestLoggingDecorator" level="INFO">
        <appender-ref ref="ACCESSLOG"/>
    </logger>
```
