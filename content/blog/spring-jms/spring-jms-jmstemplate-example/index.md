---
title: "Spring JMS JmsTemplate Example"
summary: "A detailed step-by-step tutorial on how to use JmsTemplate in combination with Spring JMS and Spring Boot."
url: /spring-jms-jmstemplate-example.html
date: 2018-12-13
lastmod: 2019-05-30
tags: ["posts", "spring", "spring jms"]
draft: false
---

Today I'm going to show you how to use the [Spring JmsTemplate](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/core/JmsTemplate.html).

The best part?

You're going to see a detailed example to get you up and running in record time.

So without further ado, let's get started…

{{< alert "lightbulb" >}}
If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{< ref "/tutorials/spring-jms-tutorials" >}}) page.
{{< /alert >}}

## What is Spring JmsTemplate?

The [JmsTemplate](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/integration.html#jms-jmstemplate) is a central class from the Spring core package.

It simplifies the use of [JMS](https://en.wikipedia.org/wiki/Java_Message_Service) and gets rid of boilerplate code. It handles the creation and release of JMS resources when sending or receiving messages.

Let's create a code sample that shows how to configure the Spring `JmsTemplate`. We will send an order message to an `order` queue and then synchronously receive a status message from a `status` queue.

We start from a previous [Spring JMS example with ActiveMQ]({{< ref "/blog/spring-jms/spring-jms-activemq-example" >}}).

## General Project Overview

We will use the following tools/frameworks:

* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.15
* Maven 3.6

Our project has the following directory structure:

![spring jms jmstemplate maven project](spring-jms-jmstemplate-maven-project.png)

## Configuring the JmsTemplate

The `JmsTemplate` was [originally designed](https://bsnyderblog.blogspot.com/2010/02/using-spring-jmstemplate-to-send-jms.html) to be used with a [J2EE container](https://docs.oracle.com/cd/E17802_01/j2ee/j2ee/1.4/docs/tutorial-update6/doc/Overview3.html). It was the responsibility of the container to provide the necessary pooling of the JMS resources (connections, sessions, consumers and producers).

With the development of frameworks like [Spring Boot](https://spring.io/projects/spring-boot) a different solution was needed. As caching of JMS resources was no longer handled by the container.

This is where the [CachingConnectionFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/connection/CachingConnectionFactory.html) comes into play. It wraps a JMS provider's connection to provide caching of sessions, connections, and producers. It also handles automatic connection recovery.

By default, it uses a single session to create many connections. If you need to scale further, you can also specify the number of sessions to cache using the `sessionCacheSize` property.

Configuring a `CachingConnectionFactory` is quite simple. Pass a `ConnectionFactory` and don't forget to set the `sessionCacheSize` if you need to scale.

The `JmsTemplate` requires a reference to a ConnectionFactory. In this example, we pass the `CachingConnectionFactory`.

> If you run on a [J2EE runtime](https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition#Certified_referencing_runtimes) obtain the `ConnectionFactory` from the application's environment naming context via JNDI.

There are a number of items that are set by default:

* Dynamic destination creation is set to `queues`.
* JMS Sessions are "not transacted" and "auto-acknowledged".
* The template uses a `DynamicDestinationResolver` and a [SimpleMessageConverter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/support/converter/SimpleMessageConverter.html).

In addition, we set the `defaultDestination` to which messages should be sent. And as we will also receive messages using the `JmsTemplate` we also set the `receiveTimeout`.

For more information check the [JmsTemplate documentation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/core/JmsTemplate.html).

> If you use [Spring JMS autoconfiguration]({{< ref "/blog/spring-jms/spring-jms-annotations-example" >}}) you can use the [Spring Boot JMS application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) (# JMS section) to set the above options.

``` java
package com.codenotfound.jms;

import javax.jms.Destination;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.activemq.command.ActiveMQQueue;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.connection.CachingConnectionFactory;
import org.springframework.jms.core.JmsTemplate;

@Configuration
public class SenderConfig {

  @Value("${activemq.broker-url}")
  private String brokerUrl;

  @Value("${destination.order}")
  private String orderDestination;

  @Value("${destination.status}")
  private String statusDestination;

  @Bean
  public ActiveMQConnectionFactory senderConnectionFactory() {
    ActiveMQConnectionFactory activeMQConnectionFactory =
        new ActiveMQConnectionFactory();
    activeMQConnectionFactory.setBrokerURL(brokerUrl);

    return activeMQConnectionFactory;
  }

  @Bean
  public CachingConnectionFactory cachingConnectionFactory() {
    CachingConnectionFactory cachingConnectionFactory =
        new CachingConnectionFactory(senderConnectionFactory());
    cachingConnectionFactory.setSessionCacheSize(10);

    return cachingConnectionFactory;
  }

  @Bean
  public Destination orderDestination() {
    return new ActiveMQQueue(orderDestination);
  }

  @Bean
  public Destination statusDestination() {
    return new ActiveMQQueue(statusDestination);
  }

  @Bean
  public JmsTemplate orderJmsTemplate() {
    JmsTemplate jmsTemplate =
        new JmsTemplate(cachingConnectionFactory());
    jmsTemplate.setDefaultDestination(orderDestination());
    jmsTemplate.setReceiveTimeout(5000);

    return jmsTemplate;
  }
}
```

## Using JmsTemplate to Send and Receive Messages

Sending messages using the `JmsTemplate` can be done in two ways:

1. _Using send(messageCreator)_: The `MessageCreator` callback interface creates the JMS message.
2. _Using convertAndSend(message, messagePostProcessor)_: The `MessageConverter` assigned to the `JmsTemplate` creates the JMS message. The `MessagePostProcessor` allows for further modification of the message after it has been processed by the converter.

We will use the second approach to send a simple order message.

Create a `Sender` class and auto-wire the `JmsTemplate`.

Define a `sendOrder()` method that takes an `orderNumber` String as parameter. The `convertAndSend()` method on the template will take the String and convert it into a JMS `TextMessage`.

We use a `MessagePostProcessor` to retrieve the `JMSMessageID`. This is a JMS header field that contains the unique identifier of the message. We will use this value to select the corresponding status message that the `Receiver` will return.

In the `receiveOrderStatus()` method we use the `receiveSelectedAndConvert()` method to synchronously receive a status message. This means that this method will block until it receives the message.

We use a [Spring JMS message selector]({{< ref "/blog/spring-jms/spring-jms-message-selector-example" >}}) to receive the status for the order previously sent.

``` java
package com.codenotfound.jms;

import java.util.concurrent.atomic.AtomicReference;
import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.Message;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;

@Component
public class Sender {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Sender.class);

  @Autowired
  private Destination statusDestination;

  @Autowired
  private JmsTemplate jmsTemplate;

  public String sendOrder(String orderNumber) throws JMSException {
    final AtomicReference<Message> message = new AtomicReference<>();

    jmsTemplate.convertAndSend(orderNumber, messagePostProcessor -> {
      message.set(messagePostProcessor);
      return messagePostProcessor;
    });

    String messageId = message.get().getJMSMessageID();
    LOGGER.info("sending OrderNumber='{}' with MessageId='{}'",
        orderNumber, messageId);

    return messageId;
  }

  public String receiveOrderStatus(String correlationId) {
    String status = (String) jmsTemplate.receiveSelectedAndConvert(
        statusDestination,
        "JMSCorrelationID = '" + correlationId + "'");
    LOGGER.info("receive Status='{}' for CorrelationId='{}'", status,
        correlationId);

    return status;
  }
}
```

In the `Receiver` class we configure a `JmsListener` to receive the order message.

We then create a status message on which we apply the [JMS Message ID Pattern](https://docs.oracle.com/cd/E13171_01/alsb/docs25/interopjms/MsgIDPatternforJMS.html#wp1047503). We copy the message ID from the request to the correlation ID of the response.

This time we use the `send()` method on the `JmsTemplate` to send back the status message. In the `MessageCreator` we create a `TextMessage` and set a simple `Accepted` String as response.

Note that we reuse the same `JmsTemplate` in both `Sender` and `Receiver` class.

> You can configure a single `JmsTemplate` instance and then safely inject this shared reference into multiple collaborators since the template is thread-safe.

``` java
package com.codenotfound.jms;

import javax.jms.Destination;
import javax.jms.TextMessage;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.support.JmsHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

  @Autowired
  private Destination statusDestination;

  @Autowired
  private JmsTemplate jmsTemplate;

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Receiver.class);

  @JmsListener(destination = "${destination.order}")
  public void receiveOrder(String orderNumber,
      @Header(JmsHeaders.MESSAGE_ID) String messageId) {
    LOGGER.info("received OrderNumber='{}' with MessageId='{}'",
        orderNumber, messageId);

    LOGGER.info("sending Status='Accepted' with CorrelationId='{}'",
        messageId);

    jmsTemplate.send(statusDestination, messageCreator -> {
      TextMessage message =
          messageCreator.createTextMessage("Accepted");
      message.setJMSCorrelationID(messageId);
      return message;
    });
  }
}
```

## Testing the JmsTemplate

To test the setup we adapt the existing test case.

We use the `Sender` to send out an order message. We then receive the status message and check that it contains the `Accepted` state.

``` java
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;
import org.apache.activemq.junit.EmbeddedActiveMQBroker;
import org.junit.ClassRule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;
import com.codenotfound.jms.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringJmsApplicationTest {

  @ClassRule
  public static EmbeddedActiveMQBroker broker =
      new EmbeddedActiveMQBroker();

  @Autowired
  private Sender sender;

  @Test
  public void testReceive() throws Exception {
    String correlationId = sender.sendOrder("order-001");
    String status = sender.receiveOrderStatus(correlationId);

    assertThat(status).isEqualTo("Accepted");
  }
}
```

Open a command prompt in the project root directory and run the test case.

``` bash
mvn test
```

In the output logs, we can see that the order and status messages are received.

``` bash
 .   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.5.RELEASE)

2019-05-30 13:52:57.496  INFO 1968 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 1968 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-jmstemplate)
2019-05-30 13:52:57.497  INFO 1968 --- [           main] c.codenotfound.SpringJmsApplicationTest  : No active profile set, falling back to default profiles: default
2019-05-30 13:52:58.430  INFO 1968 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Started SpringJmsApplicationTest in 1.264 seconds (JVM running for 2.78)
2019-05-30 13:52:58.798  INFO 1968 --- [           main] com.codenotfound.jms.Sender              : sending OrderNumber='order-001' with MessageId='ID:DESKTOP-2RB3C1U-59197-1559217176825-8:1:1:1:1'
2019-05-30 13:52:58.897  INFO 1968 --- [enerContainer-3] com.codenotfound.jms.Receiver            : received OrderNumber='order-001' with MessageId='ID:DESKTOP-2RB3C1U-59197-1559217176825-8:1:1:1:1'
2019-05-30 13:52:58.897  INFO 1968 --- [enerContainer-3] com.codenotfound.jms.Receiver            : sending Status='Accepted' with CorrelationId='ID:DESKTOP-2RB3C1U-59197-1559217176825-8:1:1:1:1'
2019-05-30 13:52:58.903  INFO 1968 --- [           main] com.codenotfound.jms.Sender              : receive Status='Accepted' for CorrelationId='ID:DESKTOP-2RB3C1U-59197-1559217176825-8:1:1:1:1'
2019-05-30 13:52:59.923  INFO 1968 --- [           main] o.a.a.junit.EmbeddedActiveMQBroker       : Stopping Embedded ActiveMQ Broker: embedded-broker
2019-05-30 13:52:59.928  INFO 1968 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://embedded-broker stopped
2019-05-30 13:52:59.928  INFO 1968 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-59197-1559217176825-0:1) is shutting down
2019-05-30 13:52:59.935  INFO 1968 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-59197-1559217176825-0:1) uptime 3.334 seconds
2019-05-30 13:52:59.936  INFO 1968 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-59197-1559217176825-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.931 s - in com.codenotfound.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.074 s
[INFO] Finished at: 2019-05-30T13:53:00+02:00
[INFO] ------------------------------------------------------------------------
```

---

{{< alert "github" >}}
If you would like to run the above code sample you can get the [full source code on GitHub](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-message-selector).
{{< /alert >}}

In this tutorial, you learned how to configure the `JmsTemplate` to send and receive JMS messages.

If you have any questions.

Or if you enjoyed reading.

Drop a line below!
