Apollo HTTPサービス
===================

[![Maven Central](https://img.shields.io/maven-central/v/com.spotify/apollo-parent.svg)](https://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.spotify%22%20apollo*)

`apollo-http-service`ライブラリはApolloモジュールの小さなバンドルです。それは[apollo-api](../apollo-api) と[apollo-core](../apollo-core)の両方を組み込み、完全なサービスを作るための[jetty-http-server](../modules/jetty-http-server)と[okhttp-client](../modules/okhttp-client)と共にその2つを結びます。それはロギング性能を与えるためにSLF4Jの実装として`logback-classic`も加えます。

Apollo HTTPサービスはあなたのバックエンドサービスを始めることを必要であるものを与えます。ここで、例えば、私たちが、`Ping::init`関数で定義し、`"ping"`と名付けたサービスをブートし、`args`変数を通して渡されたコマンドライン引数を扱うために`HttpService`と対話します：

```java
public static void main(String... args) throws LoadingException {
  HttpService.boot(Ping::init, "ping", args);
}
```

[HttpService](src/main/java/com/spotify/apollo/httpservice/HttpService.java)クラスは`apollo-api`と共に`apollo-core`と他のモジュールが十分な機能的なサービスをビルドするためにどのように一緒に来るかを示す良い例です。あなたは[/modules](../modules)の下でそれぞれのディレクトリで様々なモジュールによるドキュメントを理解することができます。

最小の骨組みプロジェクト
========================

```plain
.
├── pom.xml
└── src/
    └── main/
        ├── java/
        │   └── com/
        │       └── example/
        │           └── Ping.java
        └── resources/
            └── ping.conf
```

`./src/main/java/com/example/Ping.java`
```java
package com.example;

import com.spotify.apollo.AppInit;
import com.spotify.apollo.Environment;
import com.spotify.apollo.route.Route;
import com.spotify.apollo.httpservice.LoadingException;
import com.spotify.apollo.httpservice.HttpService;

public final class Ping {

  /**
   * The main entry point of the java process which will delegate to
   * {@link HttpService#boot(AppInit, String, String...)}.
   *
   * @param args  program arguments passed in from the command line
   * @throws LoadingException if anything goes wrong during the service boot sequence
   */
  public static void main(String... args) throws LoadingException {
    HttpService.boot(Ping::init, "ping", args);
  }

  /**
   * An implementation of the {@link AppInit} functional interface which simply sets up a
   * "hello world" handler on the root route "/".
   *
   * @param environment  The Apollo {@link Environment} that the service is in.
   */
  static void init(Environment environment) {
    environment.routingEngine()
        .registerAutoRoute(Route.sync("GET", "/ping", requestContext -> "pong"));
  }
}
```

`./src/main/resources/ping.conf`
```
# Configuration for http interface
http.server.port = 8080
http.server.port = ${?HTTP_PORT}
```

コンフィグレーションをどのように管理すべきかのより詳しく知りたいのなら、[Apollo Core](../apollo-core)、、[logback-classic](http://logback.qos.ch/)ドキュメントと[Typesafe Config](https://github.com/typesafehub/config)ドキュメントをみてください。

### Maven

`./pom.xml`
```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <name>Simple Ping Service</name>
    <groupId>com.example</groupId>
    <artifactId>ping</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <mainClass>com.example.Ping</mainClass>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.spotify</groupId>
                <artifactId>apollo-bom</artifactId>
                <version>1.2.4</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

   <dependencies>
        <dependency>
            <groupId>com.spotify</groupId>
            <artifactId>apollo-http-service</artifactId>
        </dependency>
   </dependencies>

   <build>
       <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <compilerArgs>
                        <compilerArg>-Xlint:all</compilerArg>
                    </compilerArgs>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.10</version>
                <executions>
                    <execution>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <useBaseVersion>false</useBaseVersion>
                    <overWriteReleases>false</overWriteReleases>
                    <overWriteSnapshots>true</overWriteSnapshots>
                    <includeScope>runtime</includeScope>
                    <outputDirectory>${project.build.directory}/lib</outputDirectory>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <archive>
                        <manifest>
                          <addClasspath>true</addClasspath>
                          <classpathPrefix>lib/</classpathPrefix>
                          <mainClass>${mainClass}</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

Mavenコンフィグレーションは少し長々としているのでもう少し説明をする必要があります：

* 私たちは自分のメインクラスを参照するために`mainClass`と名付けたプロパティを使います。これはあとで使います。
* Under the `dependencyManagement`の下で私たちは`apollo-bom`アーキテクチャを通して全てのApolloアーティファクトバージョンをインポートします。管理依存をインポートすることに関してのより詳しい情報は[Maven documentation](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies)をみてください。
* 私たちはJDK 8を対象とするコンパイラプラグインを設定します。
* 私たちは`${project.build.directory}/lib`の中にある全てのランタイム依存jarをコピーするために`maven-dependency-plugin`を設定します。メインアーキファクトから参照しました。
* 私たちは自分のメインクラスを使う`MainClass`エントリと一緒に`lib/`を付けて、マニフェストにクラスパスjarを加えるために`maven-jar-plugin`を設定します。

コンパイルと起動
===============
```
mvn package
java -jar target/ping.jar
```

`curl`によるリクエストを試す
```
$ curl http://localhost:8080/ping
pong
```

ロギング
=======

[おおよその](../modules/jetty-http-server/src/main/java/com/spotify/apollo/http/server/CombinedFormatLogger.java)Apache HTTPD 'combined'フォーマットをログするデフォルトの実装を使うために、HTTPサービスは受信リクエストとこれらのレスポンスをログします。

アクセスログファイルにこれを送るために、同様のコンフィグレーションを使います：

```
    <appender name="ACCESSLOG" class="ch.qos.logback.core.FileAppender">
        <file>/path/to/access.log</file>
        <encoder>
            <pattern>%msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.spotify.apollo.http.server.CombinedFormatLogger" level="INFO">
        <appender-ref ref="ACCESSLOG"/>
    </logger>
```

[Guice optional injections](https://github.com/google/guice/wiki/Injections#optional-injections)を使うロギング実装をカスタマイズするために、Guice optional injectionsのラインに沿って何かします：

```
public class MyModule extends AbstractModule {
  // ...
  protected void configure() {
    bind(com.spotify.apollo.http.server.RequestOutcomeConsumer.class).toInstance(new MyLogger());
  }
}
```
