---
title: "JAXB - Unmarshal XML String into Java Object"
summary: "A code sample on how to unmarshal an XML String into a Java Object using JAXB."
url: /jaxb-unmarshal-xml-string-into-java-object.html
date: 2013-07-31
lastmod: 2013-07-31
tags: ["posts", "jaxb"]
draft: false
aliases:
  - /2013/07/jaxb-unmarshal-xml-string.html
  - /2013/07/jaxb-unmarshal-an-xml-string.html
  - /2013/07/jaxb-unmarshalling-xml-string.html
  - /2013/07/jaxb-unmarshal-xml-string-into-java-object.html
---

When trying to unmarshal XML to a Java object using JAXB you might want to pass the XML as a `String`. However, the `unmarshal()` method of the `Unmarshaller` interface does not support passing an XML String.

Following code sample illustrates how to solve this.

Wrap the XML String in a `StringReader` object and pass this to the `unmarshal()` method as shown below.

> Note that an even [shorter one-liner notation](http://stackoverflow.com/a/9794300/4201470) is also possible.

``` java
  public static Car unmarshal(String xml) throws JAXBException {
    JAXBContext jaxbContext = JAXBContext.newInstance(Car.class);
    Unmarshaller jaxbUnmarshaller = jaxbContext.createUnmarshaller();

    StringReader reader = new StringReader(xml);
    Car car = (Car) jaxbUnmarshaller.unmarshal(reader);

    LOGGER.info(car.toString());
    return car;
  }
```

Next, pass below string representation of an XML to the above `unmarshal()` method.

``` java
xml = "<?xml version=\"1.0\" encoding=\"UTF-8\" "
  + "standalone=\"yes\"?>"
  + "<ns2:Car xmlns:ns2=\"com.codenotfound.jaxb.model\" id=\"ABC-123\">"
  + "<make>Passat</make>"
  + "<manufacturer>Volkswagen</manufacturer></ns2:Car>";
```

JAXB is now able to unmarshal the `StringReader` object and the result is the following:

``` bash
Car [make=Passat, manufacturer=Volkswagen, id=ABC-123]
```

{{< alert "github" >}}
If you would like to run the above code sample you can get the [full source code on GitHub](https://github.com/code-not-found/jaxb/tree/master/jaxb-unmarshal-string).
{{< /alert >}}

This concludes our short example on how to unmarshal an XML `String`. If you found this post helpful or have any questions or remarks, please leave a comment.
