---
title: "Spring JMS Message Selector Example"
summary: "A detailed step-by-step tutorial on how to implement a message selector using Spring JMS and Spring Boot."
url: /spring-jms-message-selector-example.html
date: 2018-12-12
lastmod: 2019-05-30
tags: ["posts", "spring", "spring jms"]
draft: false
---

In this post I'm going to show you how to configure a [JMS message selector](https://docs.oracle.com/cd/E19798-01/821-1841/bncer/index.html) using [Spring JMS](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/integration.html#jms).

You'll also see how to add information to a message so that you can select it.

So here we go.

{{< alert "lightbulb" >}}
If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{< ref "/tutorials/spring-jms-tutorials" >}}) page.
{{< /alert >}}

## What is a JMS Message Selector?

If your messaging application needs to **filter the messages it receives**, you can use a JMS message selector. It allows a message consumer to specify the messages it is interested in.

A selector is a String that contains an expression. The syntax of the expression is based on a subset of the [SQL92](https://en.wikipedia.org/wiki/SQL-92) conditional expression syntax.

Let's create an example to show how all of this works. We start from a previous [Spring JMS configuration]({{< ref "/blog/spring-jms/spring-jms-annotations-example" >}}) example.

We then modify the `Receiver` so that it receives high and low priority messages with different listeners. In the `Sender` we add a [JMS property]({{< ref "/blog/java-jms/jms-message-types-properties-overview#jms-message-properties" >}}) on which we can filter.

## General Project Overview

We will use the following tools/frameworks:

* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.15
* Maven 3.6

Our project has the following directory structure:

![spring jms selector converter maven project](spring-jms-message-selector-maven-project.png)

## Add a JMS Message Selector to a Listener

On the `@JmsListener` there is an optional message `selector` property you can define.

We create two listeners in the `Receiver`: one for high priority messages and one for low priority messages. Selection is done based on a `priority` JMS property that we will set in the `Sender`.

Note that a message consumer receives only messages whose headers and properties match the selector.

> A message selector cannot select messages on the basis of the content of the message body.

``` java
package com.codenotfound.jms;

import java.util.concurrent.CountDownLatch;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(2);

  public CountDownLatch getLatch() {
    return latch;
  }

  @JmsListener(destination = "${queue.boot}",
      selector = "priority = 'high'")
  public void receiveHigh(String message) {
    LOGGER.info("received high priority message='{}'", message);
    latch.countDown();
  }

  @JmsListener(destination = "${queue.boot}",
      selector = "priority = 'low'")
  public void receiveLow(String message) {
    LOGGER.info("received low priority message='{}'", message);
    latch.countDown();
  }
}
```

The `JmsTemplate` by default converts a String into a `TextMessage` using the `SimpleMessageConverter`. For more information on this you can check the [Spring JMS MessageConverter example]({{< ref "/blog/spring-jms/spring-jms-message-converter-example" >}}).

We can use a [MessagePostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jms/core/MessagePostProcessor.html) to add a JMS property to a message after it has been processed by the converter.

Modify the `Sender` to check an `isHighPriority` parameter. If the value equals true, a `priority` property with the value `high` is set. Otherwise the property is set to `low`.

``` java
package com.codenotfound.jms;

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
  private JmsTemplate jmsTemplate;

  public void send(String destination, String message,
      boolean isHighPriority) {
    LOGGER.info("sending message='{}' with highPriority='{}'",
        message, isHighPriority);

    if (isHighPriority) {
      jmsTemplate.convertAndSend(destination, message,
          messagePostProcessor -> {
            messagePostProcessor.setStringProperty("priority",
                "high");
            return messagePostProcessor;
          });
    } else {
      jmsTemplate.convertAndSend(destination, message,
          messagePostProcessor -> {
            messagePostProcessor.setStringProperty("priority",
                "low");
            return messagePostProcessor;
          });
    }
  }
}
```

## Test the Message Filter

We modify the existing test case so that three messages are sent.

Two of them are set with a high priority and will lower the `CountDownLatch` in the `Receiver`.

``` java
package com.codenotfound.jms;

import static org.assertj.core.api.Assertions.assertThat;
import java.util.concurrent.TimeUnit;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringJmsApplicationTest {

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @Test
  public void testReceive() throws Exception {
    sender.send("selector.q", "High priority 1!", true);
    sender.send("selector.q", "Low priority 1!", false);
    sender.send("selector.q", "High priority 2!", true);

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

Open a command prompt in the root directory and start the test case.

``` bash
mvn test
```

In the output logs, we can see that the messages are received in the respective JMS Listeners.

``` bash
 .   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.5.RELEASE)

2019-05-30 17:24:00.512  INFO 13808 --- [           main] c.c.jms.SpringJmsApplicationTest         : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 13808 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-message-selector)
2019-05-30 17:24:00.514  INFO 13808 --- [           main] c.c.jms.SpringJmsApplicationTest         : No active profile set, falling back to default profiles: default
2019-05-30 17:24:01.946  INFO 13808 --- [           main] o.apache.activemq.broker.BrokerService   : Using Persistence Adapter: MemoryPersistenceAdapter
2019-05-30 17:24:02.024  INFO 13808 --- [  JMX connector] o.a.a.broker.jmx.ManagementContext       : JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
2019-05-30 17:24:02.095  INFO 13808 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59760-1559229841967-0:1) is starting
2019-05-30 17:24:02.104  INFO 13808 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59760-1559229841967-0:1) started
2019-05-30 17:24:02.104  INFO 13808 --- [           main] o.apache.activemq.broker.BrokerService   : For help or more information please see: http://activemq.apache.org
2019-05-30 17:24:02.134  INFO 13808 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost started
2019-05-30 17:24:02.184  INFO 13808 --- [           main] c.c.jms.SpringJmsApplicationTest         : Started SpringJmsApplicationTest in 2.106 seconds (JVM running for 3.165)
2019-05-30 17:24:02.515  INFO 13808 --- [           main] com.codenotfound.jms.Sender              : sending message='High priority 1!' with highPriority='true'
2019-05-30 17:24:02.533  INFO 13808 --- [           main] com.codenotfound.jms.Sender              : sending message='Low priority 1!' with highPriority='false'
2019-05-30 17:24:02.536  INFO 13808 --- [           main] com.codenotfound.jms.Sender              : sending message='High priority 2!' with highPriority='true'
2019-05-30 17:24:02.539  INFO 13808 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received low priority message='Low priority 1!'
2019-05-30 17:24:02.539  INFO 13808 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received high priority message='High priority 1!'
2019-05-30 17:24:02.542  INFO 13808 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received high priority message='High priority 2!'
2019-05-30 17:24:03.558  INFO 13808 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost stopped
2019-05-30 17:24:03.558  INFO 13808 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59760-1559229841967-0:1) is shutting down
2019-05-30 17:24:03.567  INFO 13808 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59760-1559229841967-0:1) uptime 1.885 seconds
2019-05-30 17:24:03.568  INFO 13808 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59760-1559229841967-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.161 s - in com.codenotfound.jms.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  9.207 s
[INFO] Finished at: 2019-05-30T17:24:04+02:00
[INFO] ------------------------------------------------------------------------
```

{{< alert "github" >}}
If you would like to run the above code sample you can get the [full source code on GitHub](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-message-selector).
{{< /alert >}}

Setting up a JMS message selector is straightforward when you use Spring JMS.

Make sure to leave a comment if the example was helpful.

Thanks!
