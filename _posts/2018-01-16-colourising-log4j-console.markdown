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

You need the crashub CLI and crashub shell jars (which you can add to your dependency tree) in your lib directory:

```xml
<dependency org="org.crashub" name="crash.cli" rev="1.3.2" conf="runtime->default"/>
<dependency org="org.crashub" name="crash.shell" rev="1.3.2" conf="runtime->default"/>
```

And then it's just a case of using the `highlight` pattern in your console appender.

{% highlight xml %}
<Console name="Console" target="SYSTEM_OUT">
  <PatternLayout>
{% raw %}
    <Pattern>%highlight{%d [%t] %-5level: %msg%n%throwable}{FATAL=brite white, ERROR=bright bright red, WARN=bright yellow, INFO=bright magenta, DEBUG=bright green, TRACE=bright cyan}</Pattern>
{% endraw %}
  </PatternLayout>
  <filters>
    <ThresholdFilter level="TRACE"/>
  </filters>
</Console>
{% endhighlight %}
