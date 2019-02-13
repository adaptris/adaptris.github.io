---
layout: post
title: "Colourise your log4j output"
author: amcgrath
# date: 2018-01-15 11:00
comments: false
tags: [interlok, log4j]
categories: [interlok, log4j]
published: true
description: "Making your log4j console output pretty"
keywords: "interlok"
excerpt_separator: <!-- more -->
---

When you're testing your workflows, you're often looking at the console log. I find it really useful to colourise the output so that errors and such are easily recognisable. We can do this quite easily by including a couple of additional jars into the interlok runtime and having a custom pattern.

<!-- more -->

Log4j2 colourisation automatically works on Unix style terminals (Linux / Mac for instance) so you don't need to do anything special; however on Windows, you need to include the `jansi` artefact which handles ANSI escape code in your console, so you basically need to include that as a dependency in your dependency management.

```xml
<dependency org="org.fusesource.jansi" name="jansi" rev="1.17.1" conf="runtime->default"/>
```

And then it's just a case of using the `highlight` pattern in your console appender.

{% highlight xml %}
<Console name="Console" target="SYSTEM_OUT">
  <PatternLayout>
{% raw %}
    <Pattern>%highlight{%d [%t] %-5level: %msg%n%throwable}{FATAL=bright white, ERROR=bright red, WARN=yellow, INFO=magenta, DEBUG=green, TRACE=cyan}</Pattern>
{% endraw %}
  </PatternLayout>
  <filters>
    <ThresholdFilter level="TRACE"/>
  </filters>
</Console>
{% endhighlight %}

Note that if you are on Windows and using log4j2 > 2.9.1, then you need to explicitly enable jansi (use _-Dlog4j.skipJansi=false_) when starting up Interlok; alternatively have a `log4j2.component.properties` file on the class path that contains the same setting e.g.

```
$ cat config/log4j2.component.properties
log4j.skipJansi=false
```


