---
title: "Apache CXF - Feature vs Interceptor"
summary: "This post explains the difference between an Apache CXF feature and an interceptor and how they are linked."
url: /apache-cxf-feature-vs-interceptor.html
date: 2015-01-08
lastmod: 2015-01-08
tags: ["posts", "apache cxf"]
draft: false
aliases:
  - /2015/01/cxf-feature-versus-interceptor.html
  - /2015/01/cxf-feature-vs-interceptor.html
  - /cxf-feature-vs-interceptor.html

---

When working with CXF you'll often see that the configuration of a certain capability can be done either via adding a feature or by adding a number of interceptors. In this post we'll explain how the two are related to each other.

## Apache CXF Interceptor

[Interceptors](https://cxf.apache.org/docs/interceptors.html) are the fundamental processing unit inside CXF. For example when a service is invoked, an `InterceptorChain` (an ordered list of interceptors) is created and invoked. Each interceptor gets a chance to do what they want with the incoming message.

This can include reading it, transforming it, processing headers, validating the message, etc. Interceptors can be configured on the interceptor chains of CXF clients, CXF servers or the CXF bus.

When for example configuring logging of all messages on the CXF bus, we need to created the respective `LoggingInInterceptor` and `LoggingOutInterceptor` interceptors and add them to the different interceptor chains (`InInterceptors`, `OutInterceptors`, `OutFaultInterceptors` and `InFaultInterceptors`) which are in turn added to the CXF bus as shown below.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:cxf="http://cxf.apache.org/core"
  xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">

  <bean id="loggingInInterceptor" class="org.apache.cxf.interceptor.LoggingInInterceptor" />
  <bean id="loggingOutInterceptor" class="org.apache.cxf.interceptor.LoggingOutInterceptor" />

  <cxf:bus>
    <cxf:inInterceptors>
       <ref bean="loggingInInterceptor" />
    </cxf:inInterceptors>
    <cxf:inFaultInterceptors>
      <ref bean="loggingInInterceptor" />
    </cxf:inFaultInterceptors>
    <cxf:outInterceptors>
      <ref bean="loggingOutInterceptor" />
    </cxf:outInterceptors>
    <cxf:outFaultInterceptors>
      <ref bean="loggingOutInterceptor" />
    </cxf:outFaultInterceptors>
  </cxf:bus>

</beans>
```

## Apache CXF Feature

A [feature](https://cxf.apache.org/docs/features.html) in CXF is a way of adding capabilities to a CXF Client, CXF Server or CXF Bus. In other words features provide a simple way to perform or configure a series of related tasks. These tasks can be things like adding a number of interceptors to a chain, configuring properties, setting up resources, etc.

For example, CXF ships with a `LoggingFeature` which does exactly the same as the configuration from the previous section. The feature encapsulates the creation of the different logging interceptors and then subsequently adding them to all the interceptor chains.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:cxf="http://cxf.apache.org/core"
    xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd">

  <bean id="loggingFeature" class="org.apache.cxf.feature.LoggingFeature" />

  <cxf:bus>
    <cxf:features>
      <ref bean="loggingFeature" />
    </cxf:features>
  </cxf:bus>

</beans>
```

## Conclusion

If you need full control when enabling a certain CXF functionality then use [interceptors](https://cxf.apache.org/docs/interceptors.html). For example when you only want to log error messages, only configure the logging interceptors on the fault chains.

If you need the standard CXF functionality then use the corresponding [feature](https://cxf.apache.org/docs/featureslist.html). For example when you want to log all messages, then configure the logging feature as the notation will be shorter/simpler.
