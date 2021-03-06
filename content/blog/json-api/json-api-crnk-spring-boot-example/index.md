---
title: "JSON API - Crnk Spring Boot Example"
summary: "A detailed step-by-step tutorial on how to provide and consume a RESTful Hello World API using JSON API, Crnk, and Spring Boot."
url: /json-api-crnk-spring-boot-example.html
date: 2017-12-16
lastmod: 2017-12-16
tags: ["posts", "json api", "crnk"]
draft: false
aliases:
  - /json-api-spring-boot-crnk-example.html
---

[JSON API](http://jsonapi.org/) is a specification for building APIs using JSON. It details how clients should request resources to be fetched or modified, and how servers should respond to those requests. JSON API is designed to minimize both the number of requests and the amount of data transmitted between clients and servers.

[Crnk](http://www.crnk.io/) is an implementation of the JSON API specification and recommendations in Java to facilitate building RESTful applications. It is an additional layer that can be plugged on top of existing server-side Java implementations in order to provide easy [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) support. Crnk defines resources which can be shared over a RESTful interface and a repository for handling them.

> Crnk has its roots in the [Katharsis framework](http://katharsis.io/). However it is now [a new project with its own name](https://www.reddit.com/r/java/comments/6hs0n8/crnkio_10_released_crank_up_rest_development/dj0w13h/).

The below tutorial details how to configure and build a RESTful Hello World API that complies with the JSON API specification. Frameworks used are Crnk, OkHttp, and Spring Boot.

Tools used:

* Crnk 2.4
* OkHttp 3.6
* Spring Boot 1.5
* Maven 3.5

## General Project Setup

The example will be built and run using [Apache Maven](https://maven.apache.org/). In order to use the Crnk framework, we need to include the `crnk-core` dependency. As Spring Boot will be used for running the server part we also need to include `crnk-spring`. For implementing the client part the `crnk-client` dependency is needed.

Crnk aims to have as little dependencies as possible, as such a HTTP client library is not included by default. It needs to be [provided on the classpath where it will be automatically picked up](http://www.crnk.io/releases/stable/documentation/#_client) by the framework. Both [OkHttp](https://square.github.io/okhttp) and [Apache Http Client](https://hc.apache.org/httpcomponents-client-ga/index.html) are supported. For this example we will use the `okhttp` library.

Running and testing of the example is based on the `spring-boot-starter-web` and `spring-boot-starter-test` [Spring Boot starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters). The former is needed as we will be exposing the Hello World API using the `@RestController` annotation.

The `spring-boot-maven-plugin` is used to spin up an embedded Tomcat server that will host the RESTful Hello World API.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>json-api-crnk-helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>json-api-crnk-helloworld</name>
  <description>JSON API - Spring Boot Crnk Example</description>
  <url>https://www.codenotfound.com/json-api-crnk-spring-boot-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>
    <crnk.version>2.4.20171211095351</crnk.version>
    <okhttp.version>3.9.1</okhttp.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <!-- crnk -->
    <dependency>
      <groupId>io.crnk</groupId>
      <artifactId>crnk-core</artifactId>
      <version>${crnk.version}</version>
    </dependency>
    <dependency>
      <groupId>io.crnk</groupId>
      <artifactId>crnk-spring</artifactId>
      <version>${crnk.version}</version>
    </dependency>
    <dependency>
      <groupId>io.crnk</groupId>
      <artifactId>crnk-client</artifactId>
      <version>${crnk.version}</version>
    </dependency>
    <!-- okhttp -->
    <dependency>
      <groupId>com.squareup.okhttp3</groupId>
      <artifactId>okhttp</artifactId>
      <version>${okhttp.version}</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

Spring Boot is used in order to make a stand-alone Crnk example application that we can "just run". The `SpringCrnkApplication` class contains the `main()` method that uses Spring Boot's `SpringApplication.run()` method to launch the application.

The `@SpringBootApplication` annotation is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.

``` java
package com.codenotfound.crnk;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringCrnkApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringCrnkApplication.class, args);
  }
}
```

## Creating the Domain Model

We start by defining a simple model that will represent a `Greeting` resource. It contains an `id` in addition to the actual greeting `content`. We also need to define the constructors and getters/setters for the two fields.

The `@JsonApiResource` annotation [defines a resource](http://www.crnk.io/releases/stable/documentation/#_jsonapiresource). It requires the type parameter to be set which will be used to form the URI and populate the [type field](http://jsonapi.org/format/#document-resource-objects) which is passed as part of the resource object.

> According to JSON API standard, the name defined in type can be either plural or singular. However, the same value should be used consistently throughout an implementation.

The `@JsonApiId` [defines a field which will be used as an identifier](http://www.crnk.io/releases/stable/documentation/#_jsonapiid) of the Greeting resource. Each resource requires this annotation to be present on a field which is of primitive type or a type that implements the `Serializable` interface.

``` java
package com.codenotfound.crnk.domain.model;

import io.crnk.core.resource.annotations.JsonApiId;
import io.crnk.core.resource.annotations.JsonApiResource;

@JsonApiResource(type = "greetings")
public class Greeting {

  @JsonApiId
  private long id;

  private String content;

  public Greeting() {
    super();
  }

  public Greeting(long id, String content) {
    this.id = id;
    this.content = content;
  }

  public long getId() {
    return id;
  }

  public void setId(long id) {
    this.id = id;
  }

  public String getContent() {
    return content;
  }

  public void setContent(String content) {
    this.content = content;
  }

  @Override
  public String toString() {
    return "greeting[id=" + id + ", content=" + content + "]";
  }
}
```

## Creating the Repository

To allow Crnk to operate on defined resources, a special type of class called [repository](http://www.crnk.io/releases/stable/documentation/#_repositories) needs to be created. Crnk will scan for these classes and using annotations it will discover the available methods.

For our `Greeting` resource we will create a `GreetingRepositoryImpl` which extends the `ResourceRepositoryBase` implementation of the `ResourceRepositoryV2` repository interface.
The [ResourceRepositoryBase](https://github.com/crnk-project/crnk-framework/blob/master/crnk-core/src/main/java/io/crnk/core/repository/ResourceRepositoryBase.java) is a base class that takes care of some boiler-plate code like for example implementing `findOne()` and `findAll()`.

> Note that when extending the `ResourceRepositoryBase` only `findAll()` needs to be implemented to have a working read-only repository.

In this example, we will store the greeting resources in a simple `HashMap` and add a "Hello World!" greeting as a resource via the constructor.

Crnk passes JSON API query parameters to repositories through a `QuerySpec` parameter. The implementation of the `findAll()` uses the `apply()` method which evaluates the querySpec against the provided list and returns the result.

> Note that the `@Component` annotation is needed so that Crnk is able to find the repository.

``` java
package com.codenotfound.crnk.domain.repository;

import java.util.HashMap;
import java.util.Map;

import org.springframework.stereotype.Component;

import com.codenotfound.crnk.domain.model.Greeting;

import io.crnk.core.queryspec.QuerySpec;
import io.crnk.core.repository.ResourceRepositoryBase;
import io.crnk.core.resource.list.ResourceList;

@Component
public class GreetingRepositoryImpl extends ResourceRepositoryBase<Greeting, Long> {

  private Map<Long, Greeting> greetings = new HashMap<>();

  public GreetingRepositoryImpl() {
    super(Greeting.class);

    greetings.put(1L, new Greeting(1L, "Hello World!"));
  }

  @Override
  public synchronized ResourceList<Greeting> findAll(QuerySpec querySpec) {
    return querySpec.apply(greetings.values());
  }
}
```

## Setting up the Server

Crnk comes with [out-of-the-box support for Spring Boot](http://www.crnk.io/releases/stable/documentation/#_integration_with_spring_and_string_boot). The entry point is a `CrnkConfig` class which configures Crnk using Spring properties. Additionally, we have to make sure that each repository is annotated with `@Component` (as we did with the above `GreetingRepositoryImpl`).

Spring's `@RestController` annotation is used to mark the `CrnkController` class as a controller for handling HTTP requests. The `CrnkConfigV3` import will setup and expose the resource endpoints based on auto-scanning for specific annotations in addition to some properties which we will see further below.

Crnk also ships with a `ResourceRegistry` which holds information on all the registered repositories. We expose this information on `/resources-info` by iterating over all the entries in the auto-wired `ResourceRegistry`.

``` java
package com.codenotfound.crnk.server;

import java.util.HashMap;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Import;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import io.crnk.core.engine.registry.RegistryEntry;
import io.crnk.core.engine.registry.ResourceRegistry;
import io.crnk.spring.boot.v3.CrnkConfigV3;

@RestController
@Import({CrnkConfigV3.class})
public class CrnkController {

  @Autowired
  private ResourceRegistry resourceRegistry;

  @RequestMapping("/resources-info")
  public Map<String, String> getResources() {
    Map<String, String> result = new HashMap<>();
    // Add all resources
    for (RegistryEntry entry : resourceRegistry.getResources()) {
      result.put(entry.getResourceInformation().getResourceType(),
          resourceRegistry.getResourceUrl(entry.getResourceInformation()));
    }

    return result;
  }
}
```

In addition to the above auto-configuration, a number of items are configured using the `application.yml` properties file:

* The `crnk:resourcePackage` property specifies which package(s) should be searched to find models and repositories used by the core and exception mappers. Multiple packages can be passed by specifying a comma-separated string of packages.
* The `crnk:domainName` property specifies the domain name as well as protocol and optionally port number to be used when building links objects in responses. The value must not end with a backslash (/).
* The `crnk:pathPrefix` property defines the default prefix of the URL path used when building link objects in responses and when performing method matching.

``` yml
crnk:
  resourcePackage: com.codenotfound.crnk.domain
  domainName: http://localhost:9090/codenotfound
  pathPrefix: /api

server:
  context-path: /codenotfound
  port: 9090
```

## Setting up the Client

Crnk includes a [Java client](http://www.crnk.io/releases/stable/documentation/#_client) which allows to communicate with JSON-API compliant servers. To start using the client just create an instance of `CrnkClient` and pass the service URL. Then use the client to create a repository that gives access to the different resource CRUD operations.

In below `GreetingClient` we have created a `findOne()` method that returns a single `Greeting` based on the identifier.

``` java
package com.codenotfound.crnk.client;

import javax.annotation.PostConstruct;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import com.codenotfound.crnk.domain.model.Greeting;

import io.crnk.client.CrnkClient;
import io.crnk.core.queryspec.QuerySpec;
import io.crnk.core.repository.ResourceRepositoryV2;

@Component
public class GreetingClient {

  private static final Logger LOGGER = LoggerFactory.getLogger(GreetingClient.class);

  private CrnkClient crnkClient = new CrnkClient("http://localhost:9090/codenotfound/api");
  private ResourceRepositoryV2<Greeting, Long> resourceRepositoryV2;

  @PostConstruct
  public void init() {
    resourceRepositoryV2 = crnkClient.getRepositoryForType(Greeting.class);
  }

  public Greeting findOne(long id) {
    Greeting result = resourceRepositoryV2.findOne(id, new QuerySpec(Greeting.class));

    LOGGER.info("found {}", result.toString());
    return result;
  }
}
```

## Testing the Hello World JSON API

Let's test our greeting resource! If you are running Google Chrome you can simply do this from the browser. Make sure you start the application by starting up Spring Boot:

``` text
mvn spring-boot:run
```

Next open following URL which should show all our registered resources (in this example the greetings resource).

``` text
http://localhost:9090/codenotfound/resources-info
```

![hello world resource registry](hello-world-resource-registry.png)

Go ahead and use the greetings endpoint as next URL. This will show us all the available greetings.

``` text
http://localhost:9090/codenotfound/api/greetings
```

![hello world greetings](hello-world-greetings.png)

For the details of our Hello World resource we enter below URL.

``` text
http://localhost:9090/codenotfound/api/greetings/1
```

![hello world greeting 1](hello-world-greeting-1.png)

> Notice that above flow is fully driven using hypermedia!

To wrap up we will also add a simple `SpringCrnkApplicationTest` unit test case in which we lookup the Hello World greeting resource using its identifier.

``` java
package com.codenotfound.crnk;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.crnk.client.GreetingClient;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class SpringCrnkApplicationTest {

  @Autowired
  GreetingClient greetingClient;

  @Test
  public void testFindOne() {
    assertThat(greetingClient.findOne(1L).getContent()).isEqualTo("Hello World!");
  }
}
```

Triggering the test case is done using following maven command.

``` bash
mvn test
```

The output should be as shown below.

``` bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

20:34:21.843 [main] INFO  c.c.crnk.SpringCrnkApplicationTest - Starting SpringCrnkApplicationTest on cnf-pc with PID 4220 (started by CodeNotFound in c:\code\json-api\json-api-crnk-helloworld)
20:34:21.846 [main] INFO  c.c.crnk.SpringCrnkApplicationTest - No active profile set, falling back to default profiles: default
20:34:24.289 [main] INFO  c.c.crnk.SpringCrnkApplicationTest - Started SpringCrnkApplicationTest in 2.738 seconds (JVM running for 3.354)
20:34:24.623 [main] INFO  c.c.crnk.client.GreetingClient - found greeting[id=1, content=Hello World!]
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.164 sec - in com.codenotfound.crnk.SpringCrnkApplicationTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 5.299 s
[INFO] Finished at: 2017-12-17T20:34:24+01:00
[INFO] Final Memory: 19M/227M
[INFO] ------------------------------------------------------------------------
```

{{< alert "github" >}}
If you would like to run the above code sample you can get the [full source code on GitHub](https://github.com/code-not-found/json-api/tree/master/json-api-crnk-helloworld).
{{< /alert >}}

This wraps up the Crnk JSON API tutorial in which we developed a simple Hello World example from scratch and exposed it via a RESTful API.

Let me know if you liked this tutorial or in case you have a question, please leave a comment below.
