---
title:             "Simplified Branching"
description:       "The proposed replacement for branching-service-collection is available"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            quotidian-ennui
excerpt_separator: <!-- more -->
---

3.9.0 sees the promotion of the _interlok-config-conditional_ package into the main trunk; as a result you can use a number of new services where you might previously have had to use `branching-service-collection`; the ultimate aim is to remove the need to use `branching-service-collection` over the next few releases as we achieve feature parity between the likes of `if-then-otherwise`; along with its associated conditions; and `branching-service-collection`. There will always be cases where branching-service-collection may be more appropriate; but we think we've covered most of the use-cases with the new classes.


<!-- more -->

The new classes render well when you are looking at them in the XML view; the UI is having to undergo some surgery in order to catch up. This is a quick overview of the broad classes that are now available.

## if-then-otherwise

This is essentially your classic _if_ statement; if the condition is true, then execute the associated service, otherwise do the other service. There isn't support _elseif_ or similar, since once in while we do have opinions...

```xml
<if>
  <condition class="some-condition"/>
  <then>
    <service class="service-list"/>
  </then>
  <otherwise>
    <service class="service-list"/>
  </otherwise>
</if>
```

## switch

This is of course a _switch_ construct; since you can nest different types of condition; it can fulfil the use-case where you absolutely have an `elseif`. Each `case` is evaluated in order; with the special condition _case-default_ always returning true.

```xml
<switch>
  <case>
    <condition class="some-condition"/>
    <service class="service-list"/>
  </case>
  <case>
    <condition class="some-other-condition"/>
    <service class="service-list"/>
  </case>
  <case>
    <condition class="yet-another-condition"/>
    <service class="service-list"/>
  </case>
  <case>
    <condition class="case-default"/>
    <service class="service-list"/>
  </case>
```

## while / do-while

This mirrors the while / do-while construct available in most programming languages and follows the same semantics: a while loop may never execute, but a do-while loop will always execute at least once. Bear in mind that the same message instance is operated on within the loop; you might end up overwriting the payload with the last loop iteration. To avoid infinite loops, we have a `max-loops` setting which defaults to 10 (but can be explicitly changed to -1 if you do want the possibility of infinite loops)

```xml
<while>
  <condition class="some-condition"/>
  <then>
    <service class="service-list"/>
  </then>
</while>

<do-while>
  <condition class="some-condition"/>
  <then>
    <service class="service-list"/>
  </then>
</do-while>
```

In 3.9.1; there will pluggable behaviour that you can configure when the maximum number of loops is exceeded.

## Conditions

There already a number of conditions available; with more in development to reach feature parity with the number of branching-services available; the Condition interface is designed to be extensible as required. There is already support for a Javascript condition, and a beanshell expression funtion, so very complex conditions are already possible with those constructs.