# Apollo API

このApolloライブラリはあなたが自分のサービスルートと自分のリクエスト/リプライハンドラであるツールを与えます。いくつかの概要文書によって、以下をご覧ください：

* [AppInit & Environment](docs/app-init-environment.md)
* [Routing Engine](docs/routing-engine.md)
* [Routes](docs/routes.md)
* [Response](docs/response.md)
* [Middleware](docs/middleware.md)

## 最低限のことをちょうど与えます

### 1. `src/main/java/com/spotify/Small.java`

```java
package com.spotify;

import com.spotify.apollo.AppInit;
import com.spotify.apollo.Environment;
import com.spotify.apollo.route.Route;
import com.spotify.apollo.httpservice.LoadingException;
import com.spotify.apollo.httpservice.HttpService;

public final class Small {

  /**
   * The main entry point of the java process which will delegate to
   * {@link HttpService#boot(AppInit, String, String...)}.
   *
   * @param args  program arguments passed in from the command line
   * @throws LoadingException if anything goes wrong during the service boot sequence
   */
  public static void main(String... args) throws LoadingException {
    HttpService.boot(Small::init, "small", args);
  }

  /**
   * An implementation of the {@link AppInit} functional interface which simply sets
   * up a "hello world" handler on the root route "/".
   *
   * @param environment  The Apollo {@link Environment} that the service is in.
   */
  static void init(Environment environment) {
    environment.routingEngine()
        .registerAutoRoute(Route.sync("GET", "/", requestContext -> "hello world"));
  }
}
```

### 2. Mavenでビルド！

あなたの`pom.xml`の`apollo-http-service`の依存を加えます。あなたがビルド構成をあとで必要としてからバージョンのビルドプロパティを使います。

```xml
<properties>
    <apollo.version>1.1.0</apollo.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.spotify</groupId>
            <artifactId>apollo-bom</artifactId>
            <version>${apollo.version}</version>
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
```

`lib/`の下に依存jarを置いてクラスパスによってjarを作るためのビルドを準備してください。

```xml
<build>
    <finalName>${project.artifactId}</finalName>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.3</version>
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
            <version>2.5</version>
            <configuration>
                <archive>
                    <addMavenDescriptor>true</addMavenDescriptor>
                    <manifest>
                        <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                        <addDefaultSpecificationEntries>true</addDefaultSpecificationEntries>
                        <addClasspath>true</addClasspath>
                        <classpathPrefix>lib/</classpathPrefix>
                        <mainClass>com.spotify.Small</mainClass>
                    </manifest>
                    <manifestEntries>
                        <X-Spotify-Apollo-Version>${apollo.version}</X-Spotify-Apollo-Version>
                    </manifestEntries>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 3. ビルドと起動！

```
mvn package
java -jar target/small.jar -Dhttp.server.port=8080
```

### 4. Curl！

```
curl http://localhost:8080/
> hello world
```
