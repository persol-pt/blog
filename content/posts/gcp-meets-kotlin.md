---
title: "GCP Meets Kotlin"
date: 2017-12-15T17:16:57+09:00
tags: ["GCP", "GAE", "Kotlin"]
categories: ["keitaro"]
---

この記事は [Google Cloud Platform Advent Calendar 2017](https://qiita.com/advent-calendar/2017/gcp) の15日目の記事です。<br/>
<br/>
## はじめに
今年盛り上がったプログラミング言語の一つがKotlinであるというのは衆目の一致するところだと思います。<br/>
Google I/OでAndoridアプリ開発の公式言語としてサポートされることが発表されたのを始め、様々なニュースが話題になっていました。<br/>
<br/>
Kotlinの特徴を簡単にいうと、

- JVM上で動作するプログラミング言語
- Javaとの100%互換
- Null安全、型安全

となります。<br/>
<br/>
最近ではサーバサイドのプログラミング言語としても注目され採用事例も増えて来ています(弊社でも採用を検討しています)<br/>
<br/>
特徴であげた「Javaとの100%互換」があるため、Javaを使うGoogle Cloud Platformの各種サービスでもKotlinで実装することが出来ます。<br/>
その実装例を書いていきたいと思います。<br/>
<br/>
## Google App Engine [Standard enviroment]
[コード](https://github.com/keitaro1020/gcp-meets-kotlin/tree/master/gae-standard) <br/>
<br/>
GAE StandardはGoogleが用意した専用コンテナで動作します。使用できる言語が限られていたり制約は多いのですが、その分開発や運用が楽です。<br/>
どのフレームワークでも多分問題なく動作するとは思いますが、今回はSprinngBoot(2.0.0.M7)で作りました。<br/>
<br/>
### プログラムソース

* メインクラス

warで実行するためにSpringBootServletInitializerクラスを継承し、configureメソッドをオーバーライドします。<br/>
```java
@SpringBootApplication
class Application : SpringBootServletInitializer() {
    @Bean
    fun cloudDatastoreService(): DatastoreService{
        return DatastoreServiceFactory.getDatastoreService()
    }

    override fun configure(builder: SpringApplicationBuilder): SpringApplicationBuilder {
        return builder.sources(Application::class.java)
    }
}

fun main(args: Array<String>) {
    runApplication<Application>(*args)
}
```

* Datastoreへのアクセス

Datastoreに接続するには「[Google Cloud Java Client](https://github.com/GoogleCloudPlatform/google-cloud-java/tree/master/google-cloud-datastore)」か[appengine API](https://cloud.google.com/appengine/docs/standard/java/javadoc/)」があります。<br/>
Standardの場合は「Google Cloud Java Client」を利用するとローカル環境で実行した際に
```
Caused by: java.lang.IllegalStateException: Must use project ID as app ID if project ID is provided.
```
と例外が出てしまい、ローカルのエミュレータに繋ぐことが出来ないため「appengine API」を使用しました。<br/>
（どちらを使用してもGAE上での動作は問題ありませんでした。ローカル環境での実行時にローカルDatastore（エミュレータ）に接続できるかどうかの差です）<br/>
<br/>
### 設定ファイル

* buiid.gradle

GAE Standardは、warファイルをデプロイしJetty9上で動作させます。<br/>
そのため、

 - spring-boot-starter-tomcat を exclude して spring-boot-starter-jetty を適用
 - warpluginを使用

しています
```java
(略)

apply plugin: 'kotlin'
apply plugin: "kotlin-spring"
apply plugin: "org.springframework.boot"
apply plugin: 'io.spring.dependency-management'
apply plugin: "com.google.cloud.tools.appengine"
apply plugin: 'war'
apply plugin: 'org.akhikhl.gretty'

(略)

dependencies {
    compile "javax.servlet:javax.servlet-api:3.1.0"

    compile("org.springframework.boot:spring-boot-starter-freemarker")
    compile("org.springframework.boot:spring-boot-starter-web"){
        exclude module: "spring-boot-starter-tomcat"
    }
    providedRuntime 'org.springframework.boot:spring-boot-starter-jetty'

    compile("org.jetbrains.kotlin:kotlin-stdlib-jre8:${kotlinVersion}")
    compile("org.jetbrains.kotlin:kotlin-reflect:${kotlinVersion}")

    compile("com.fasterxml.jackson.module:jackson-module-kotlin:2.9.2")

    compile("com.google.appengine:appengine:+")
    compile("com.google.appengine:appengine-api-1.0-sdk:+")

    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

* appengine-web.xml

runtimeはjava8を使用しますプロジェクト内のサービスを分ける場合は&lt;module&gt;に指定します
```xml
<?xml version="1.0" encoding="utf-8"?>
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
    <runtime>java8</runtime>
    <threadsafe>true</threadsafe>
    <module>gae-standard</module>
    <sessions-enabled>true</sessions-enabled>
    <system-properties>
        <property name="java.util.logging.config.file" value="WEB-INF/logging.properties"/>
    </system-properties>
    <automatic-scaling>
        <min-idle-instances>1</min-idle-instances>
        <max-idle-instances>1</max-idle-instances>
    </automatic-scaling>
</appengine-web-app>
```

### その他

* ローカル環境での実行

```sh
> ./gradlew appengineRun
```

* デプロイ

```sh
> ./gradlew appengineDeploy
```

## Google App Engine [Flexible enviroment]
[コード](https://github.com/keitaro1020/gcp-meets-kotlin/tree/master/gae-flexible) <br/>
<br/>
GAE FlexibleはGAEを名乗ってはいますが実質GCEです。Dockerコンテナ上でアプリを動作させるためStandardに比べて出来ることは多いですが、起動時間が長かったり、Standardと違って無料枠での利用が出来ない等のデメリットがあります。<br/>
<br/>
### プログラムソース

* メインクラス

```Java
package sample

import com.google.cloud.datastore.Datastore
import com.google.cloud.datastore.DatastoreOptions
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.context.annotation.Bean

@SpringBootApplication
class Application {
    @Bean
    fun cloudDatastoreService(): Datastore {
        return DatastoreOptions.getDefaultInstance().service
    }
}

fun main(args: Array<String>) {
    runApplication<Application>(*args)
}
```

* Datastoreへのアクセス

「[Google Cloud Java Client](https://github.com/GoogleCloudPlatform/google-cloud-java/tree/master/google-cloud-datastore)」を使用します。<br/>
<br/>

### 設定ファイル

* buiid.gradle

Dockerコンテナ上でアプリを動作させるため、warファイルを作成せずに実行可能jarを動作させることが出来ます。<br/>
そのためStandardで利用していたwarpluginは必要ありません。

```java
(略)
apply plugin: 'kotlin'
apply plugin: "kotlin-spring"
apply plugin: "org.springframework.boot"
apply plugin: 'io.spring.dependency-management'
apply plugin: "com.google.cloud.tools.appengine"

(略)

dependencies {
    compile "javax.servlet:javax.servlet-api:3.1.0"

    compile("org.springframework.boot:spring-boot-starter-freemarker")
    compile("org.springframework.boot:spring-boot-starter-web"){
        exclude module: "spring-boot-starter-tomcat"
    }
    compile('org.springframework.boot:spring-boot-starter-jetty')

    compile("org.jetbrains.kotlin:kotlin-stdlib-jre8:${kotlinVersion}")
    compile("org.jetbrains.kotlin:kotlin-reflect:${kotlinVersion}")

    compile("com.fasterxml.jackson.module:jackson-module-kotlin:2.9.2")

    compile("com.google.appengine:appengine:+")
    compile("com.google.cloud:google-cloud-datastore:1.8.0")

    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

* Dockerfile

```
FROM gcr.io/google_appengine/openjdk8
VOLUME /tmp
ADD gae-flexible-0.0.1-SNAPSHOT.jar app.jar
CMD [ "java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

* app.yaml

```yaml
runtime: custom
env: flex
service: gae-flexible

automatic_scaling:
  min_num_instances: 1
  max_num_instances: 1
```

### その他

* ローカ環境での実行

```sh
> ./gradlew bootRun
```

* デプロイ
デプロイは非常に時間がかかります。GAE Standardが1分程度でデプロイ出来るのに比べ、Flexibleでは6〜8分程度かかります。

```sh
> ./gradlew appengineDeploy
```

## Cloud Endpoints + GAE standard(endpoints framework)
[コード](https://github.com/keitaro1020/gcp-meets-kotlin/tree/master/endpoints) <br/>
<br/>
EndpointsはAPIの保護・監視・管理のためのAPIゲートウェイです。<br/>
Firebase Authentication、Google 認証、Auth0などのOpenId Connectに対応した認証を利用できるなど、APIの開発効率を上げることができます。<br/>

EndPointsは下記の組み合わせで利用できます。<br/>

- OpenAPI : swagger等で作成した定義ファイルをEndpointsにデプロイし、以下の環境に実装されたAPIを利用
  - Google App Engine Flexible
  - Google Compute Engine
  - Google Kubernetes Engine
- Endpoints Frameworks : App Engine用のFrameworkで、以下の環境に実装されたAPIを利用
  - Google App Engine Standard (Java)
  - Google App Engine Standard (Python)
- gRPC
  - Google Compute Engine
  - Google Kubernetes Engine

[公式のサイト](https://cloud.google.com/endpoints/docs/?hl=ja)にはgRPC + GAE Flexibleが利用できるように書いてありますがこれは誤りのようです(英語版は修正されている)<br/>
<br/>
### プログラムソース

* API実装

APIに付けたアノテーションにEndpointsの定義を指定します。このアノテーションからAPIドキュメントやクライアントライブラリを生成できます。

```Java
@Api(
  name = "sample",
  version = "v1",
  namespace =
    ApiNamespace(
      ownerDomain = "sample",
      ownerName = "sample",
      packagePath = ""
    )
)
class MessagesApi {

    val messageRepository: MessagesRepository

    init {
        messageRepository = DatastoreMessagesRepository(DatastoreServiceFactory.getDatastoreService())
    }

    @ApiMethod(path = "messages/{id}", httpMethod = "get")
    fun getMessage(@Named("id")id: Long) : Message? {
        return messageRepository.findById(id = id)
    }

    @ApiMethod(path = "messages", httpMethod = "get")
    fun getMessages() : List<Message> {
        return messageRepository.findAll()
    }

    @ApiMethod(path = "messages", httpMethod = HttpMethod.POST)
    fun createMessage(message: Message) : Message? {
        messageRepository.create(message = message)
        return messageRepository.findById(id = message.id)
    }

}
```

### 設定ファイル
* appengine-web.xml

env-variablesにEndpointsサービス名を指定します。

```xml
<?xml version="1.0" encoding="utf-8"?>
<appengine-web-app xmlns="http://appengine.google.com/ns/1.0">
    <threadsafe>true</threadsafe>
    <basic-scaling>
        <max-instances>2</max-instances>
    </basic-scaling>
    <system-properties>
        <property name="java.util.logging.config.file" value="WEB-INF/logging.properties"/>
    </system-properties>
    <env-variables>
        <env-var name="ENDPOINTS_SERVICE_NAME" value="${endpoints.project.id}.appspot.com" />
    </env-variables>
</appengine-web-app>
```

* web.xml

servlet にAPIのクラスを指定し、filterにEndpointsサービス名を指定します。

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"  version="2.5">
    <!-- Wrap the backend with Endpoints Frameworks v2. -->
    <servlet>
        <servlet-name>EndpointsServlet</servlet-name>
        <servlet-class>com.google.api.server.spi.EndpointsServlet</servlet-class>
        <init-param>
            <param-name>services</param-name>
            <param-value>sample.MessagesApi</param-value>
        </init-param>
    </servlet>
    <!-- Route API method requests to the backend. -->
    <servlet-mapping>
        <servlet-name>EndpointsServlet</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>

    <!-- [START api_management] -->
    <!-- Add a filter that fetches the service config from service management. -->
    <filter>
        <filter-name>endpoints-api-configuration</filter-name>
        <filter-class>com.google.api.control.ServiceManagementConfigFilter</filter-class>
    </filter>

    <!-- Add a filter that performs Endpoints logging and monitoring. -->
    <filter>
        <filter-name>endpoints-api-controller</filter-name>
        <filter-class>com.google.api.control.extensions.appengine.GoogleAppEngineControlFilter</filter-class>
        <init-param>
            <param-name>endpoints.projectId</param-name>
            <param-value>${endpoints.project.id}</param-value>
        </init-param>
        <init-param>
            <param-name>endpoints.serviceName</param-name>
            <param-value>${endpoints.project.id}.appspot.com</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>endpoints-api-configuration</filter-name>
        <servlet-name>EndpointsServlet</servlet-name>
    </filter-mapping>

    <filter-mapping>
        <filter-name>endpoints-api-controller</filter-name>
        <servlet-name>EndpointsServlet</servlet-name>
    </filter-mapping>
    <!-- [END api_management] -->
</web-app>
```

### その他

* デプロイ

作成したAPIからAPIドキュメントを作成します。<br/>
ドキュメントは`build/endpointsOpenApiDocs/openapi.json`に作成されます。

```sh
> ./gradlew endpointsOpenApiDocs
```

APIドキュメントをEndpointsにデプロイします

```sh
> gcloud endpoints services deploy build/endpointsOpenApiDocs/openapi.json
```

APIをデプロイします

```sh
> ./gradlew appengineDeploy
```

## さいごに

Kotlinが「Javaとの100%互換」を謳っているだけあって、実装すること自体は難しくありません。 ~~GCPの情報が少ない方が辛い~~ <br/>
今回は取り上げませんでしたが、同じくJavaを使うCloud DataflowでもKotlinでの実装を行うことが出来ます（[コード](https://github.com/keitaro1020/gcp-meets-kotlin/tree/master/dataflow))<br/>
<br/>
簡潔で安全に書けるKotlinとGCPの組み合わせ、これからの選択肢としていかがでしょうか？<br/>
<br/>


