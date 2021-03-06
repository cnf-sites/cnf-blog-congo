---
title: "GraphQL Java - Spring Boot Example"
summary: "A detailed step-by-step tutorial on how to implement a Hello World GraphQL API using Java, Spring Boot, and Maven."
url: /graphql-java-spring-boot-example.html
date: 2018-03-15
lastmod: 2018-03-15
tags: ["posts", "graphql", "spring boot"]
draft: false
---

[GraphQL](http://graphql.org/) is a query language for APIs. It provides a complete and understandable description of the data in your API and gives clients the power to ask for exactly what they need and nothing more.

GraphQL [was developed internally by Facebook](https://en.wikipedia.org/wiki/GraphQL) in 2012 before being publicly released in 2015. Many [different programming languages support GraphQL](http://graphql.org/code/), this example focuses on [Java](https://java.com/).

In the following tutorial, we will configure, build and run a Hello World GraphQL API using Java, Spring Boot, and Maven.

## General Project Setup

Tools used:

* Graphql Spring Boot 3.10
* Graphql Java Tools 4.3
* Spring Boot 1.5
* Maven 3.5

We will be building and running our example using [Apache Maven](https://maven.apache.org/). Shown below is the XML representation of our Maven project in a POM file. It contains the needed dependencies for compiling and running the example.

In order to configure and expose the Hello World GraphQL API endpoint, we will use the [Spring Boot](https://projects.spring.io/spring-boot/) project.

To facilitate the management of the different Spring dependencies, [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters) are used which are a set of convenient dependency descriptors that you can include in your application. We include the `spring-boot-starter-web` dependency which will automatically setup an embedded Apache Tomcat that will host the GraphQL API endpoint.

The [GraphQL-Java](https://github.com/graphql-java/graphql-java) project provides a Java implementation for GraphQL. It contains a [graphql-spring-boot](https://github.com/graphql-java/graphql-spring-boot) project that offers a `graphql-spring-boot-starter` which will auto-configure a GraphQL Servlet that is accessible on `/graphql`.

In addition, the GraphQL starter will automatically create a GraphQL schema by parsing all GraphQL schema files found on the classpath. In order for this to work the `graphql-java-tools` library also needs to be added as a dependency.

[GraphiQL](https://github.com/graphql/graphiql) provides an interface for editing and testing your GraphQL API. It becomes accessible at `/graphiql` if the `graphiql-spring-boot-starter` is added as a dependency.

The plugins section includes the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "uber-jar". This will also allow us to startup our GraphQL API server via a Maven command.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>graphql-spring-boot</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>graphql-spring-boot</name>
  <description>GraphQL - Spring Boot Example</description>
  <url>https://www.codenotfound.com/graphql-spring-boot-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.10.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>

    <graphql-spring-boot-starter.version>3.10.0</graphql-spring-boot-starter.version>
    <graphql-java-tools.version>4.3.0</graphql-java-tools.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- graphql-java -->
    <dependency>
      <groupId>com.graphql-java</groupId>
      <artifactId>graphql-spring-boot-starter</artifactId>
      <version>${graphql-spring-boot-starter.version}</version>
    </dependency>
    <dependency>
      <groupId>com.graphql-java</groupId>
      <artifactId>graphql-java-tools</artifactId>
      <version>${graphql-java-tools.version}</version>
    </dependency>
    <dependency>
      <groupId>com.graphql-java</groupId>
      <artifactId>graphiql-spring-boot-starter</artifactId>
      <version>${graphql-spring-boot-starter.version}</version>
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

## Creating a GraphQL Schema

A schema defines your GraphQL API by specifying each field that can be interacted with. More specifically the schema will contain:

* Types and fields on those types
* Operations to get data ([queries](http://graphql.org/learn/queries/)) or to create, update, and delete data ([mutations](http://graphql.org/learn/queries/#mutations)).

`graphql-java` offers two different ways of defining the schema:

1. Programmatically as Java code
2. Via a GraphQL schema language often referred to as Interface Definition Language (IDL) or Schema Definition Language (SDL)

In this example, we will use the second option. In this way we don't rely on a specific programming language syntax.

We start by defining a `Greeting` type that has two fields: an `id` and a `message`. Each field can be a [scalar type](http://graphql.org/learn/schema/#scalar-types) (Int, Float, String, Boolean and ID) an [enumeration type](http://graphql.org/learn/schema/#enumeration-types) or a reference to another type.

> Note that adding an exclamation mark indicates a type [cannot be null](http://graphql.org/learn/schema/#lists-and-non-null).

When creating a GraphQL schema there must be exactly one root query, and up to one root mutation.

For this tutorial we create a `getGreeting` Query that will allow us to retrieve a `Greeting` by passing an `id`. We also define a `newGreeting` Mutation that will enable us to create a `Greeting` by passing a `message`.

``` js
type Greeting {
  id: ID!
  message: String!
}

type Query {
  getGreeting(id: ID!): Greeting
}

type Mutation {
  newGreeting(message: String!): Greeting!
}
```

The above schema is stored in a `.graphqls` file, which can be placed anywhere on the classpath. In this example it is located in `src/main/resources/greeting.graphqls`.

## Data Classes and Resolvers

As mentioned, the `graphql-spring-boot-starter` will automatically try to build a GraphQL schema by parsing all GraphQL schema files found on the classpath.

This is done by mapping fields on your GraphQL types to methods and properties on available Java objects. For most scalar fields, a plain old Java object (POJO) with variables and/or getter methods is enough to describe the data to GraphQL.

For our `Greeting` type we create a corresponding `Greeting` class with `id` and `message` variables. The GraphQL Java Tools library will now be able to automatically map the GraphQL type to the Java object.

``` java
package com.codenotfound.model;

public class Greeting {

  private String id;

  private String message;

  public Greeting() {}

  public Greeting(String id, String message) {
    this.id = id;
    this.message = message;
  }

  public String getId() {
    return id;
  }

  public void setId(String id) {
    this.id = id;
  }

  public String getMessage() {
    return message;
  }

  public void setMessage(String message) {
    this.message = message;
  }
}
```

More complex fields often need more complex lookup methods (repositories, connections, etc) that cannot be automatically derived from data classes. GraphQL uses the concept of [Resolvers](http://graphql.org/learn/execution/#root-fields-resolvers) to account for these situations.

In this example the `getGreeting` Query and `newGreeting` Mutation types are root GraphQL objects that don't have any associated data classes. When using GraphQL Java Tools we need to use resolvers implementing `GraphQLQueryResolver` and `GraphQLMutationResolver` in order to map to fields for the respective root types.

Let's start by defining a `GreetingQuery` that implements the `GraphQLQueryResolver` interface. We define a `getGreeting()` method that matches the signature of our `getGreeting` Query. The `Greeting` is fetched from an [autowired](https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/html/beans.html#beans-factory-autowire) `GreetingRepository` that we will define further below.

> Note we use the `@Component` annotation so that Spring will [automatically import the bean](https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/html/beans.html#beans-scanning-autodetection) into the container.

``` java
package com.codenotfound.graphql.resolver;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import com.codenotfound.model.Greeting;
import com.codenotfound.repository.GreetingRepository;
import com.coxautodev.graphql.tools.GraphQLQueryResolver;

@Component
public class GreetingQuery implements GraphQLQueryResolver {

  @Autowired
  private GreetingRepository greetingRepository;

  public Greeting getGreeting(String id) {
    return greetingRepository.find(id);
  }
}
```

Similar we define a `GreetingMutation` that implements the `GraphQLMutationResolver` interface. The class contains a `newGreeting()` method that matches the signature of our `newGreeting` Mutation. The `Greeting` is saved in a `GreetingRepository` that is auto-wired.

``` java
package com.codenotfound.graphql.resolver;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import com.codenotfound.model.Greeting;
import com.codenotfound.repository.GreetingRepository;
import com.coxautodev.graphql.tools.GraphQLMutationResolver;

@Component
public class GreetingMutation implements GraphQLMutationResolver {

  @Autowired
  private GreetingRepository greetingRepository;

  public Greeting newGreeting(String message) {
    Greeting greeting = new Greeting();
    greeting.setMessage(message);

    return greetingRepository.save(greeting);
  }
}
```

In a real-life application, data will typically be fetched from a database or via an existing API. For this setup, we define a basic `GreetingRepository` that simply stores the `Greeting` objects in an in-memory `HashMap`.

``` java
package com.codenotfound.repository;

import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

import org.springframework.stereotype.Component;

import com.codenotfound.model.Greeting;

@Component
public class GreetingRepository {

  private Map<String, Greeting> greetings;

  public GreetingRepository() {
    greetings = new HashMap<>();
  }

  public Greeting save(Greeting greeting) {
    String id = UUID.randomUUID().toString();

    greetings.put(id, greeting);
    greeting.setId(id);

    return greeting;
  }

  public Greeting find(String id) {
    return greetings.get(id);
  }
}
```

## Testing with GraphiQL

Let's test our Hello World GrapQL API by starting up Spring Boot using following Maven command:

``` bash
mvn spring-boot:run
```

The easiest way to explore and interact with a GraphQL API is by using the GraphiQL GUI: [http://localhost:8080/graphiql](http://localhost:8080/graphiql)

![graphiql welcome](graphiql-welcome.png)

We start by creating a new greeting by using the `newGreeting` Mutation.

Notice how we first specify that we want a new greeting to be created with a "Hello Spring Boot GrapQL!" message. We then declare that we want the `id` and `message` fields of the greeting to be returned once it is created.

``` js
mutation {
  newGreeting(message: "Hello Spring Boot GrapQL!") {
    id
    message
  }
}
```

Click on the "Execute Query" button (play icon) and the result should appear on the right side as shown below. 

![graphiql new greeting](graphiql-new-greeting.png)

Now that we have created a greeting we can fetch it using the `getGreeting` Query. As the `id` is auto-generated simply copy/paste the value from the response of the above `newGreeting` Mutation.

Again we first specify the Query followed by the fields we would like to see returned.

``` js
{
  getGreeting(id: "34746b4e-465c-4d9b-a038676285da2990") {
    id
    message
  }
}
```

![graphiql get greeting](graphiql-get-greeting.png)

We will finish by illustrating that a client can ask for exactly what they need and nothing more. Let's execute the query of the previous step but this time we specify we only need the `message` to be returned.

``` js
{
  getGreeting(id: "34746b4e-465c-4d9b-a038676285da2990") {
    message
  }
}
```

![graphiql get greeting message](graphiql-get-greeting-message.png)

{{< alert "github" >}}
If you would like to run the above code sample you can get the [full source code on GitHub](https://github.com/code-not-found/graphql-java/tree/master/graphql-java-spring-boot).
{{< /alert >}}

GraphQL offers an alternative approach to building APIs that focus on consumer ease of use. Tools like GraphQL Java and Spring Boot enable you to quickly get a GraphQL API up and running.

Hope you enjoyed this tutorial. Drop a line in case you have some questions or would like to see another example.
