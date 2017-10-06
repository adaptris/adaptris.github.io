---
layout: post
title: "POJOs from XML value elements"
author: gerco
date: 2017-10-06 13:21
comments: false
tags: [interlok, configuration]
categories: [interlok, configuration]
published: true
description: "How to create 'complex' objects from simple XML configuration"
keywords: "interlok, continuous deployment"
excerpt_separator: <!-- more -->
---

Sometimes - when you are developing an Interlok service - you want the user to input a list of objects in the 
configuration. Some of these objects will be complex and some will only have a single value. By default, the 
XML comes out looking terrible (all the “simple” objects will have an extra tag inside) but you want the XML 
to look as simple as possible, like so:

<!-- more -->

```xml
<object-metadata>
  <s3-serverside-encryption>
    <enabled>true</enabled>
    <algorithm>AES_256</algorithm>
  </s3-serverside-encryption>
  <s3-http-expires-date>
    <time-to-live>
      <unit>MINUTES</unit>
      <interval>10</interval>
    </time-to-live>
  </s3-http-expires-date>
  <s3-content-disposition>ads</s3-content-disposition>
  <s3-content-language>en</s3-content-language>
  <s3-content-type>application/octet-stream</s3-content-type>
  <s3-expiration-rule-id>123</s3-expiration-rule-id>
</object-metadata>
```

You see that some of these are complex elements (`s3-serverside-encryption`) and some are simple value elements 
(`s3-content-language`). In reality, each of these tags deserialize to a POJO with one or more fields. The 
`s3-http-expires-date` class looks like you might expect:

```java
@XStreamAlias("s3-http-expires-date")
public class S3HttpExpiresDate extends S3ObjectMetadata {
  @NotNull @Valid
  private TimeInterval timeToLive;

  ... more code ...
}
```

But the `s3-content-language` class has a custom XStream converter to achieve the “value element” effect in the 
configuration XML:

```java
@XStreamAlias("s3-content-language")
@XStreamConverter(value = ToAttributedValueConverter.class, strings = { "contentLanguage" })
public class S3ContentLanguage extends S3ObjectMetadata {
  @NotNull @InputFieldHint(expression = true)
  private String contentLanguage;

  ... more code ...
}
```

The field named in the `strings` attribute to the `@XStreamConverter` annotation indicates the field that will be 
used as the value of the resulting XML element. Any other fields will be converted into attributes.

#### Note

At the moment, this approach breaks relaxng schema validation so if you're using that, don't do the above.
