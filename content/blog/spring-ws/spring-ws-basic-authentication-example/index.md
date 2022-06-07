---
title: "Spring WS - Basic Authentication Example"
summary: "A detailed step-by-step tutorial on how to configure basic authentication using Spring-WS and Spring Boot."
url: /spring-ws-basic-authentication-example.html
date: 2017-04-24
lastmod: 2017-04-29
tags: ["posts", "spring", "spring ws"]
draft: false
aliases:
  - /2017/04/spring-ws-basic-authentication-example.html
---

[Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) (BA) is a method for a HTTP client to provide a user name and password when making a request. There is no [confidentiality](https://en.wikipedia.org/wiki/Confidentiality) protection for the transmitted credentials. therefore it is strongly advised to use it in conjunction with HTTPS.

The credentials are provided as a HTTP header field called `'Authorization'` which is constructed as follows:

1. The username and password are combined with a single colon.

    ``` text
    codenotofound:p455w0rd
    ```

2. The resulting string is encoded into an [octet sequence](https://tools.ietf.org/html/rfc7617#section-2) and then [Base64 encoded](https://tools.ietf.org/html/rfc4648#section-4). You can use an [online Base64 decoder](https://www.base64decode.org/) to decode below value.

    ``` text
    Y29kZW5vdGZvdW5kOnA0NTV3MHJk
    ```

3. The authorization method and a space (`"Basic "`) are then put before the encoded string.

    ``` text
    Basic Y29kZW5vdGZvdW5kOnA0NTV3MHJk
    ```

Instead of writing custom code to create and check the HTTP authorization header we will configure Spring WS and Spring Boot to do the work for us. The below example illustrates how a client and server can be configured to apply basic access authentication using Spring-WS, Spring Boot, and Maven.

{{< alert "lightbulb" >}}
If you want to learn more about Spring WS - head on over to the [Spring WS tutorials]({{< ref "/tutorials/spring-ws-tutorials" >}}) page.
{{< /alert >}}

## General Project Setup

Tools used:

* Spring-WS 2.4
* Spring Security 4.2
* HttpClient 4.5
* Spring Boot 1.5
* Maven 3.5

The setup of the sample is based on a previous [Spring WS tutorial]({{< ref "/blog/spring-ws/spring-ws-wsdl-example" >}}) in which we have swapped out the basic `helloworld.wsdl` for a more generic `ticketagent.wsdl` from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex).

There are two implementations of the `WebServiceMessageSender` interface for sending messages via HTTP. The default implementation is the `HttpUrlConnectionMessageSender`, which uses the facilities provided by Java itself. The alternative is the `HttpComponentsMessageSender`, which uses the [Apache HttpComponents HttpClient](https://hc.apache.org/httpcomponents-client-ga).

We will use the `HttpComponentsMessageSender` implementation in below example as it contains more advanced and easy-to-use functionality. On GitHub, however, we have also added a [basic authentication example that uses the HttpUrlConnectionMessageSender implementation](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-basic-authentication) in case a dependency on the `HttpClient` is not desired.

In order to use the `HttpComponentsMessageSender` implementation, we need to add the Apache `httpclient` dependency to the Maven POM file.

In addition we will need the `spring-boot-starter-security` [Spring Boot starter](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters) dependency for the server setup.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-ws-basic-authentication</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-ws-basic-authentication</name>
  <description>Spring WS - Basic Authentication Example</description>
  <url>https://www.codenotfound.com/spring-ws-basic-authentication-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>
    <httpclient.version>4.5.4</httpclient.version>
    <maven-jaxb2-plugin.version>0.13.3</maven-jaxb2-plugin.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web-services</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <!-- httpclient -->
    <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
      <version>${httpclient.version}</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <!-- maven-jaxb2-plugin -->
      <plugin>
        <groupId>org.jvnet.jaxb2.maven2</groupId>
        <artifactId>maven-jaxb2-plugin</artifactId>
        <version>${maven-jaxb2-plugin.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>generate</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <schemaDirectory>${project.basedir}/src/main/resources/wsdl</schemaDirectory>
          <schemaIncludes>
            <include>*.wsdl</include>
          </schemaIncludes>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

## Setup Client Basic Authentication

In this example, we use the Apache HTTP Client, as it comes with built-in support for setting the basic authentication header. We update the `ClientConfig` class with a bean that creates an `HttpComponentsMessageSender` on which we set a `UsernamePasswordCredentials` bean. This bean will automatically create the HTTP basic authentication header.

The `@Value` annotation is used to inject the `name` and `password` values from the `application.yml` properties file shown below. These values are then set on the `UsernamePasswordCredentials` bean.

``` yaml
client:
  default-uri: http://localhost:9090/codenotfound/ws/helloworld
  user:
    name: codenotfound
    password: p455w0rd
```

We finish by setting the `HttpComponentsMessageSender` on our `WebServiceTemplate`.

``` java
package com.codenotfound.ws.client;

import org.apache.http.auth.UsernamePasswordCredentials;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.transport.http.HttpComponentsMessageSender;

@Configuration
public class ClientConfig {

  @Value("${client.default-uri}")
  private String defaultUri;

  @Value("${client.user.name}")
  private String userName;

  @Value("${client.user.password}")
  private String userPassword;

  @Bean
  Jaxb2Marshaller jaxb2Marshaller() {
    Jaxb2Marshaller jaxb2Marshaller = new Jaxb2Marshaller();
    jaxb2Marshaller.setContextPath("org.example.ticketagent");

    return jaxb2Marshaller;
  }

  @Bean
  public WebServiceTemplate webServiceTemplate() {
    WebServiceTemplate webServiceTemplate = new WebServiceTemplate();
    webServiceTemplate.setMarshaller(jaxb2Marshaller());
    webServiceTemplate.setUnmarshaller(jaxb2Marshaller());
    webServiceTemplate.setDefaultUri(defaultUri);
    // set a HttpComponentsMessageSender which provides support for basic authentication
    webServiceTemplate.setMessageSender(httpComponentsMessageSender());

    return webServiceTemplate;
  }

  @Bean
  public HttpComponentsMessageSender httpComponentsMessageSender() {
    HttpComponentsMessageSender httpComponentsMessageSender = new HttpComponentsMessageSender();
    // set the basic authorization credentials
    httpComponentsMessageSender.setCredentials(usernamePasswordCredentials());

    return httpComponentsMessageSender;
  }

  @Bean
  public UsernamePasswordCredentials usernamePasswordCredentials() {
    // pass the user name and password to be used
    return new UsernamePasswordCredentials(userName, userPassword);
  }
}
```

## Setup Server Basic Authentication

The Spring Boot security starter that was added to our Maven setup has a dependency on Spring Security. If Spring Security is on the classpath then [web applications will automatically be secured with HTTP basic authentication](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-security) on all HTTP endpoints. In other words our, `TicketAgentEndpoint` is now secured with basic auth.

The default user that will be configured has as name `'user'`. The password is randomly generated at startup (it is displayed in the startup logs).

Typically you will want to configure a custom value for the user and password, in order to do this you need to set the corresponding [Spring Boot security properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) (look for the `# SECURITY` heading) in the application properties file.

In this example we set the `'user'` to `"codenotfound"` and the `'password'` to `"p455w0rd"` in the `application.yml` properties using the YAML variant as shown below.

``` yml
security:
  user:
    name: codenotfound
    password: p455w0rd
```

## Testing the Basic Authentication Configuration

In order to test the configuration we just run the `SpringWsApplicationTests` unit test case by issuing the following Maven command.

``` bash
mvn test
```

The test case will run successfully as basic authentication is correctly configured on both sides. By default, the basic authentication header is not logged but if you want you can add some custom code in order to have [Spring-WS log all the client HTTP headers]({{< ref "/blog/spring-ws/spring-ws-log-client-server-http-headers-example" >}}).

``` bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

21:06:58.016 [main] INFO  c.c.ws.SpringWsApplicationTests - Starting SpringWsApplicationTests on cnf-pc with PID 2176 (started by CodeNotFound in c:\code\spring-ws\spring-ws-basic-authentication)
21:06:58.018 [main] INFO  c.c.ws.SpringWsApplicationTests - No active profile set, falling back to default profiles: default
21:07:01.275 [main] INFO  c.c.ws.SpringWsApplicationTests - Started SpringWsApplicationTests in 3.568 seconds (JVM running for 4.234)
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.039 sec - in com.codenotfound.ws.SpringWsApplicationTests

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7.454 s
[INFO] Finished at: 2017-04-29T21:07:01+02:00
[INFO] Final Memory: 27M/217M
[INFO] ------------------------------------------------------------------------
```

Now change the password in the `application.yml` file to a different value and rerun the test case. This time the test case will fail as a `'401 Unauthorized'` is returned by our server.

``` bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

21:52:41.786 [main] INFO  c.c.ws.SpringWsApplicationTests - Starting SpringWsApplicationTests on cnf-pc with PID 5908 (started by CodeNotFound in c:\code\spring-ws\spring-ws-basic-authentication)
21:52:41.789 [main] INFO  c.c.ws.SpringWsApplicationTests - No active profile set, falling back to default profiles: default
21:52:45.159 [main] INFO  c.c.ws.SpringWsApplicationTests - Started SpringWsApplicationTests in 3.666 seconds (JVM running for 4.281)
Tests run: 1, Failures: 0, Errors: 1, Skipped: 0, Time elapsed: 4.004 sec <<< FAILURE! - in com.codenotfound.ws.SpringWsApplicationTests
testListFlights(com.codenotfound.ws.SpringWsApplicationTests)  Time elapsed: 0.263 sec  <<< ERROR!
org.springframework.ws.client.WebServiceTransportException:  [401]
        at org.springframework.ws.client.core.WebServiceTemplate.handleError(WebServiceTemplate.java:699)
        at org.springframework.ws.client.core.WebServiceTemplate.doSendAndReceive(WebServiceTemplate.java:609)
        at org.springframework.ws.client.core.WebServiceTemplate.sendAndReceive(WebServiceTemplate.java:555)
        at org.springframework.ws.client.core.WebServiceTemplate.marshalSendAndReceive(WebServiceTemplate.java:390)
        at org.springframework.ws.client.core.WebServiceTemplate.marshalSendAndReceive(WebServiceTemplate.java:383)
        at org.springframework.ws.client.core.WebServiceTemplate.marshalSendAndReceive(WebServiceTemplate.java:373)
        at com.codenotfound.ws.client.TicketAgentClient.listFlights(TicketAgentClient.java:30)
        at com.codenotfound.ws.SpringWsApplicationTests.testListFlights(SpringWsApplicationTests.java:26)


Results :

Tests in error:
  SpringWsApplicationTests.testListFlights:26 ╗ WebServiceTransport  [401]

Tests run: 1, Failures: 0, Errors: 1, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.460 s
[INFO] Finished at: 2017-04-29T21:52:45+02:00
[INFO] Final Memory: 18M/227M
[INFO] ------------------------------------------------------------------------
```

{{< alert "github" >}}
If you would like to run the above code sample you can get the [full source code on GitHub](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-basic-authentication-httpclient).
{{< /alert >}}

Setting up basic authentication on the client side using Spring WS is pretty simple when using the Apache client. The server side is even easier when running on Spring Boot.

Drop me a line if you found the example useful. Or let me know in case of questions.
