# レスポンス

[`Response<T>`](/apollo-api/src/main/java/com/spotify/apollo/Response.java)はルートハンドラの返す型による任意のラッパーです。あなたが異なる状態を設定するかハンドラを加えるかのようなサービスリプライの追加パラメータを操作したい時に使います。

例：

```java
Response<String> handle(RequestContext requestContext) {
  String s = stringResponse();
  return Response.forPayload(s)
      .withHeader("X-Payload-Length", String.valueOf(s.length()));
}
```

```java
Response<String> handle(RequestContext requestContext) {
  String arg = requestContext.request().getParameter("arg");
  if (arg == null || arg.isEmpty()) {
    return Response.forStatus(
        Status.BAD_REQUEST.withReasonPhrase("Mandatory query parameter 'arg' is missing"));
  }

  return Response.forPayload("Your " + arg + " is valid");
}
```
