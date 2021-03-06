---
title: "Spring WS - Client Timeout Example"
summary: "A detailed step-by-step tutorial on how to set a client timeout using Spring-WS and Spring Boot."
url: /spring-ws-client-timeout-example.html
date: 2017-12-08
lastmod: 2017-12-08
tags: ["posts", "spring", "spring ws"]
draft: false
aliases:
  - /spring-ws-timeout-example.html
---

When implementing a web service client, it is a good practice to take into account the scenario where the web service call takes a long time to complete. In this case, a timeout at client side could be used in order to avoid that the client remains blocked for a significant period of time.

The following step by step tutorial illustrates an example in which we will configure a Spring-WS timeout at client side. In addition, we will show how to handle the timeout exception. The example will use Spring Boot and Maven in order to configure, build and run.

{{< alert "lightbulb" >}}
If you want to learn more about Spring WS - head on over to the [Spring WS tutorials]({{< ref "/tutorials/spring-ws-tutorials" >}}) page.
{{< /alert >}}

## General Project Setup

Tools used:

* Spring-WS 2.4
* HttpClient 4.5
* Spring Boot 1.5
* Maven 3.5

The setup of the example is based on a previous [Spring WS tutorial]({{< ref "/blog/spring-ws/spring-ws-wsdl-example" >}}) in which we have swapped out the basic `helloworld.wsdl` for a more generic `ticketagent.wsdl` from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex).

There are two implementations of the `WebServiceMessageSender` interface for sending messages via HTTP. The default implementation is the `HttpUrlConnectionMessageSender`, which uses the facilities provided by Java itself. The alternative is the `HttpComponentsMessageSender`, which uses the [Apache HttpComponents HttpClient](https://hc.apache.org/httpcomponents-client-ga).

We will use the `HttpComponentsMessageSender` implementation in below example as it contains more advanced and easy-to-use functionality. On GitHub, however, we have also added a [timeout example that uses the HttpUrlConnectionMessageSender implementation](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-client-timeout) in case a dependency on the `HttpClient` is not desired.

As the `HttpComponentsMessageSender` has a dependency on the Apache `HttpClient`, we need to add the dependency to the Maven POM file.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-ws-timeout-httpclient</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-ws-timeout-httpclient</name>
  <description>Spring WS - Timeout Example</description>
  <url>https://www.codenotfound.com/spring-ws-client-timeout-example.html</url>

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

## Setting the Template Timeout

As mentioned above, in this example we will use the `WebServiceMessageSender` implementation that uses the [Apache HttpComponents HttpClient](https://hc.apache.org/httpcomponents-client-ga/). There are two setters that allow controlling how long the client will wait. The `setConnectionTimeout()` specifies how long the client will wait before a connection to the server is successfully established. The `setReadTimeout()` configures how long the client will wait for a response once the request has been sent.

In the below example we have created a `webServiceMessageSender` bean on which we have set both timeouts with the same `timeout` value that is loaded from the `application.yml` properties file shown below.

``` yaml
client:
  default-uri: http://localhost:9090/codenotfound/ws/helloworld
  timeout: 2000
```

In order to use the `webServiceMessageSender` bean in our client we need to set it onto the `webServiceTemplate` using the `setMessageSender()` method.

``` java
package com.codenotfound.ws.client;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.transport.WebServiceMessageSender;
import org.springframework.ws.transport.http.HttpComponentsMessageSender;

@Configuration
public class ClientConfig {

  @Value("${client.default-uri}")
  private String defaultUri;

  @Value("${client.timeout}")
  private int timeout;

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
    webServiceTemplate.setMessageSender(webServiceMessageSender());

    return webServiceTemplate;
  }

  @Bean
  public WebServiceMessageSender webServiceMessageSender() {
    HttpComponentsMessageSender httpComponentsMessageSender = new HttpComponentsMessageSender();
    // timeout for creating a connection
    httpComponentsMessageSender.setConnectionTimeout(timeout);
    // when you have a connection, timeout the read blocks for
    httpComponentsMessageSender.setReadTimeout(timeout);

    return httpComponentsMessageSender;
  }
}
```

## Catching the Timeout Exception

The `WebServiceTemplate` will now throw an exception in case a timeout occurs. As such we need to handle this in our `TicketAgentClient` implementation.

We add a try/catch block around the `marshalSendAndReceive()` method in order to catch the timeout exception. If the exception occurs we log it and create an empty flight list to be returned.

``` java
package com.codenotfound.ws.client;

import java.math.BigInteger;
import java.util.ArrayList;
import java.util.List;

import javax.xml.bind.JAXBElement;

import org.example.ticketagent.ObjectFactory;
import org.example.ticketagent.TFlightsResponse;
import org.example.ticketagent.TListFlights;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.ws.client.core.WebServiceTemplate;

@Component
public class TicketAgentClient {

  private static final Logger LOGGER = LoggerFactory.getLogger(TicketAgentClient.class);

  @Autowired
  private WebServiceTemplate webServiceTemplate;

  @SuppressWarnings("unchecked")
  public List<BigInteger> listFlights() {
    ObjectFactory factory = new ObjectFactory();
    TListFlights tListFlights = factory.createTListFlights();

    JAXBElement<TListFlights> request = factory.createListFlightsRequest(tListFlights);
    JAXBElement<TFlightsResponse> response = null;

    try {
      response = (JAXBElement<TFlightsResponse>) webServiceTemplate.marshalSendAndReceive(request);
      return response.getValue().getFlightNumber();
    } catch (Exception e) {
      LOGGER.error(e.getMessage());
      // TODO handle the exception
      return new ArrayList<>();
    }
  }
}
```

## Testing the Client Timeout

To wrap up we will create a basic unit test case in which a timeout exception will be triggered on the client side. In order to achieve this, we will modify the `TicketAgentEndpoint` implementation and introduce a `sleep()` of 10 seconds before a response is returned.

``` java
package com.codenotfound.ws.endpoint;

import java.math.BigInteger;

import javax.xml.bind.JAXBElement;

import org.example.ticketagent.ObjectFactory;
import org.example.ticketagent.TFlightsResponse;
import org.example.ticketagent.TListFlights;
import org.springframework.ws.server.endpoint.annotation.Endpoint;
import org.springframework.ws.server.endpoint.annotation.PayloadRoot;
import org.springframework.ws.server.endpoint.annotation.RequestPayload;
import org.springframework.ws.server.endpoint.annotation.ResponsePayload;

@Endpoint
public class TicketAgentEndpoint {

  @PayloadRoot(namespace = "http://example.org/TicketAgent.xsd", localPart = "listFlightsRequest")
  @ResponsePayload
  public JAXBElement<TFlightsResponse> listFlights(
      @RequestPayload JAXBElement<TListFlights> request) throws InterruptedException {

    // sleep for 10 seconds so a timeout occurs
    Thread.sleep(10 * 1000);

    ObjectFactory factory = new ObjectFactory();
    TFlightsResponse tFlightsResponse = factory.createTFlightsResponse();
    tFlightsResponse.getFlightNumber().add(BigInteger.valueOf(101));

    return factory.createListFlightsResponse(tFlightsResponse);
  }
}
```

In addition, we change the existing test case to expect an empty flight list as shown below.

``` java
package com.codenotfound.ws;

import static org.assertj.core.api.Assertions.assertThat;

import java.math.BigInteger;
import java.util.List;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.ws.client.TicketAgentClient;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class SpringWsApplicationTests {

  @Autowired
  private TicketAgentClient ticketAgentClient;

  @Test
  public void testListFlights() {
    List<BigInteger> flights = ticketAgentClient.listFlights();

    assertThat(flights.size()).isEqualTo(0);
  }
}
```

We are now ready to test the timeout. Open a command prompt in the projects root folder and execute following Maven command:

``` bash
mvn test
```

The result should be a successful build during which the timeout exception error is logged as shown below:

``` bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

15:44:29.142 [main] INFO  c.c.ws.SpringWsApplicationTests - Starting SpringWsApplicationTests on cnf-pc with PID 712 (started by CodeNotFound in c:\code\spring-ws\spring-ws-timeout-httpclient)
15:44:29.145 [main] INFO  c.c.ws.SpringWsApplicationTests - No active profile set, falling back to default profiles: default
15:44:31.824 [main] INFO  c.c.ws.SpringWsApplicationTests - Started SpringWsApplicationTests in 2.988 seconds (JVM running for 3.637)
15:44:33.951 [main] ERROR c.c.ws.client.TicketAgentClient - I/O error: Read timed out; nested exception is java.net.SocketTimeoutException: Read timed out
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.228 sec - in com.codenotfound.ws.SpringWsApplicationTests

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 9.903 s
[INFO] Finished at: 2017-12-08T15:44:36+01:00
[INFO] Final Memory: 19M/228M
[INFO] ------------------------------------------------------------------------
```

{{< alert "github" >}}
If you would like to run the above code sample you can get the [full source code on GitHub](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-timeout-httpclient).
{{< /alert >}}

This concludes our example of how to configure a client timeout and catch the corresponding exception using Spring-WS and Spring Boot.

Drop a line if you enjoyed reading this post or in case you have a question you would like to ask.
