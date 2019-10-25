---
title:             "Configuration evolves, nothing stays the same"
description:       "You can cut and paste old config; but you probably should investigate new features"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            quotidian-ennui
excerpt_separator: <!-- more -->
---

One of the things that happen when you start using Interlok for any length of time is that you end up copying old configuration, templated or otherwise; it works, there aren't any deprecation warnings, and it gets the job done. In some cases, what we might have done previously is different to how we'd approach the problem now. A case in point is the building up of complex metadata values.

<!-- more -->

Previously, back in the mists of time, there was only one way to build up complex metadata; and that was through the use of the metadata-appender-service. You can still do it like this, but you end up in a situation where you define all sorts of temporary metadata keys that you might forget to get rid of; it ends up being very verbose in terms of XML bloat.

For instance; if we take a use case of _I want to build the key for an S3 upload so that the document is structured in the way YYYY/mm/dd/message-id.json so that the keys are predictable_. It's a pretty standard requirement, and one that would be handled if you were producing to the filesystem through the use of a `formatted-filename-creator` (your format string could just be `%2$tY/%2$tm/%$2td/%1$s.json`)[^1]. However uploading documents to S3 doesn't have a concept of filenames in the same way so we have to build up the metadata.

### Classic

The classic way, like a well cared for vintage car, works; is very explicit and will conceptually work in pretty much all interlok versions from 3.0 to the latest and greatest. This is somewhat a contrived example since it's obvious that we can reduce the number of services by making our _date-format_ do the whole date format dance in one step.

```xml
<add-timestamp-metadata-service>
  <metadata-key>year</metadata-key>
  <date-format-builder>
    <format>yyyy</format>
  </date-format-builder>
</add-timestamp-metadata-service>
<add-timestamp-metadata-service>
  <metadata-key>month</metadata-key>
  <date-format-builder>
    <format>MM</format>
  </date-format-builder>
</add-timestamp-metadata-service>
<add-timestamp-metadata-service>
  <metadata-key>day</metadata-key>
  <date-format-builder>
    <format>dd</format>
  </date-format-builder>
</add-timestamp-metadata-service>
<add-metadata-service>
  <metadata-element>
    <key>filetype</key>
    <value>json</value>
  </metadata-element>
  <metadata-element>
    <key>slash</key>
    <value>/</value>
  </metadata-element>
  <metadata-element>
    <key>dot</key>
    <value>.</value>
  </metadata-element>
  <metadata-element>
    <key>message_unique_id</key>
    <value>$UNIQUE_ID$</value>
  </metadata-element>
</add-metadata-service>
<metadata-appender-service>
  <result-key>s3_filename</result-key>
  <append-key>year</append-key>
  <append-key>slash</append-key>
  <append-key>month</append-key>
  <append-key>slash</append-key>
  <append-key>day</append-key>
  <append-key>slash</append-key>
  <append-key>message_unique_id</append-key>
  <append-key>dot</append-key>
  <append-key>filetype</append-key>
</metadata-appender-service>
```

So, we can see what it's doing there, it's effectively adding 7 (potentially 5) items of metadata, and then using `metadata-appender-service` to add the actual key we want which is __s3_filename__.

### Using add-formatted-metadata-service

_3.2.1_ saw the release of `add-formatted-metadata-service` which helps in the sense that you're reducing the amount of metadata that you're creating. Conceptually, it's still largely the same, but in this instance we're going to shorten the sequence, so that add-timestamp-metadata-service is only executed once.

```xml
<add-timestamp-metadata-service>
  <metadata-key>today</metadata-key>
  <date-format-builder>
    <format>yyyy/MM/dd</format>
  </date-format-builder>
</add-timestamp-metadata-service>
<add-metadata-service>
  <metadata-element>
    <key>message_unique_id</key>
    <value>$UNIQUE_ID$</value>
  </metadata-element>
</add-metadata-service>
<add-formatted-metadata-service>
  <format-string>%s/%s.json</format-string>
  <metadata-key>s3_filename</metadata-key>
  <argument-metadata-key>today</argument-metadata-key>
  <argument-metadata-key>message_unique_id</argument-metadata-key>
</add-formatted-metadata-service>
```

You still need to add the unique-id of the message as metadata since it isn't metadata; so effectively the number of services between this and _Classic_ haven't changed. There is, however, certainly less bloat in terms of text.

### Using the expression language

Resolving of metadata via an expression language was introduced in _3.6.2_. At the time of release not all services supported it, but certainly by the release of _3.6.3_ most of the metadata manipulation services had the appropriate annotation, and the UI started showing hints accordingly. This last example cuts down the number of services to 2, since you don't have to add any additional metadata at all[^2]; you can also forgo the use of actual arguments since you don't have any formatting specifiers or the like. _Note that in this example, we're using the HTML encoded form for % since otherwise we fall foul of markdown liquid syntax parsing; use a % sign._

```xml
<add-timestamp-metadata-service>
  <metadata-key>today</metadata-key>
  <date-format-builder>
    <format>yyyy/MM/dd</format>
  </date-format-builder>
</add-timestamp-metadata-service>
<add-formatted-metadata-service>
  <format-string>%message{today}/%message{&#37;uniqueId}.json</format-string>
  <metadata-key>s3_filename</metadata-key>
</add-formatted-metadata-service>
```

### Conclusion

Since we always have an eye on backwards compability and stability; what you have configured will be portable across versions for a long time. Pay attention to _WARNING_ logging in the log file which will tell you if configuration that you're using have been deprecated. If you use the UI, then you'll start getting warnings about the use of deprecated components directly in the UI now. Our deprecation policy is generally at least 2 minor releases (i.e. stuff that was deprecated in 3.5.x, is will be there in 3.7.x, but may be removed in 3.8.x).


[^1]: It is somewhat an annoyance that String.format differs slightly from SimpleDateFormat in its semantics around _M/m_, since I'm always getting it wrong
[^2]: %uniqueId resolution to give you the unique-id is recent (_3.8.0_)
