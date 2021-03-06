---
title: "JSF - Primefaces Bootstrap Theme Tutorial"
summary: "A detailed step-by-step tutorial on how to setup a Bootstrap theme using PrimeFaces, BootsFaces, Spring Boot, and Maven."
url: /jsf-primefaces-bootstrap-theme.html
date: 2018-03-06
lastmod: 2018-03-06
tags: ["posts", "primefaces", "jsf"]
draft: true
---

[Bootstrap](https://getbootstrap.com/) is an open-source toolkit for designing responsive, mobile-first projects on the web. PrimeFaces ships with an [easy to setup Bootstrap community theme](/jsf-primefaces-theme-spring-boot.html) however when comparing with the original Bootstrap theme there is a noticeable difference.

[BootsFaces](https://www.bootsfaces.net/) is a powerful and lightweight JSF framework which is based based on Bootstrap 3 and jQuery UI. As a result it is a great option when trying to combine JSF development with a Bootstrap look and feel and responsive design.

The following example shows how to setup a PrimeFaces and BootsFaces project using Spring Boot and Maven.

## General Project Setup

Tools used:

* Spring Boot 1.5
* PrimeFaces 6.1
* BootsFaces 6.1
* JoinFaces 2.4
* Maven 3.5

As the PrimeFaces community themes are not available in the [Maven central repository](http://repo1.maven.org/) we need to specify the [PrimeFaces Maven Repository](http://repository.primefaces.org) in our Maven POM file as shown below.

Once this is done we need to include the [dependency to the specific PrimeFaces theme](https://repository.primefaces.org/org/primefaces/themes/) we want to use. Alternatively, we can include all available themes by specifying the `all-themes` dependency. We will do the later in this example.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-theme-spring-boot</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>jsf-primefaces-theme-spring-boot</name>
  <description>JSF - Primefaces Theme using Spring Boot</description>
  <url>https://www.codenotfound.com/jsf-primefaces-theme-spring-boot.html</url>

  <parent>
    <groupId>org.joinfaces</groupId>
    <artifactId>jsf-spring-boot-parent</artifactId>
    <version>2.4.1</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!-- joinfaces -->
    <dependency>
      <groupId>org.joinfaces</groupId>
      <artifactId>jsf-spring-boot-starter</artifactId>
    </dependency>
    <!-- primefaces -->
    <dependency>
      <groupId>org.primefaces.themes</groupId>
      <artifactId>all-themes</artifactId>
      <version>1.0.10</version>
    </dependency>
  </dependencies>

  <repositories>
    <repository>
      <id>primefaces-maven-repository</id>
      <name>PrimeFaces Maven Repository</name>
      <url>http://repository.primefaces.org</url>
    </repository>
  </repositories>

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

We will start from a previous [JSF Spring Boot Example](/jsf-primefaces-example-spring-boot-maven.html) in which we created a greeting dialog based on a first and last name input field.

As we are using [JoinFaces](https://github.com/joinfaces/joinfaces#joinfaces) to setup Spring Boot we can use an [application property](https://github.com/joinfaces/joinfaces#jsf-properties-configuration-via-applicationproperties-or-applicationyml) in order to configure a PrimeFaces theme.

We set the `Afterdark` theme by specifying the `jsf.primefaces.theme` property in `src/main/resources/application.yml` as shown below.

``` yml
jsf:
  primefaces: 
    theme: afterdark

server:
  context-path: /codenotfound
  port: 9090
```

Let's checkout the new look of our web application by running following Maven command:

``` bash
mvn spring-boot:run
```

Once Spring Boot has started, open a web browser and enter the following URL: [http://localhost:9090/codenotfound/helloworld.xhtml](http://localhost:9090/codenotfound/helloworld.xhtml).

The below web page should now be displayed in the newly configured theme.

![jsf primefaces theme spring boot](jsf-primefaces/jsf-primefaces-theme-spring-boot.png)

{{< alert "github" >}}
If you would like to run the above code sample you can get the [full source code on GitHub](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-theme-spring-boot).
{{< /alert >}}

In this post, we illustrated how to configure a Spring Boot PrimeFaces theme.

Drop a line below if you enjoyed reading this post.
