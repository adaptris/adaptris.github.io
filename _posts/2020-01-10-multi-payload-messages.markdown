---
title:              "Multi-Payload Messages"
description:        "An introduction to multi-payload messages"
author:             albinoloverats
date:               2020-01-09
published:          true
tags:               [interlok, multi-payload, messages]
categories:         [interlok, multi-payload, messages]
keywords:           "interlok, multi-payload, messages"
excerpt_separator:  <!-- more -->
---

The Multi-Payload Adaptris Message is a new message type that supports
having multiple payloads in a single message. Each payload can be
accessed by its user-defined ID.

<!-- more -->

They are fully backwards compatible with the traditional Adaptris
Message and there are a few services to help with getting them in to
use.

The traditional message type only has one payload and therefore all
conventional services will just read and write data to the message
payload. However if there are multiple payloads available it becomes
necessary to switch between them. Multi-payload aware services will, in
future, do this as and when they please but for now there is the aptly
named Switch Payload service. This will switch to an existing payload in
the message that has the given unique ID.

![Add Payload]({{ site.baseurl }}/images/posts/multi-payload/mpm-add-payload-1.png)

![Add Payload]({{ site.baseurl }}/images/posts/multi-payload/mpm-add-payload-2.png)

Obviously before you can switch between payloads, it is necessary to
have more than one. The quickest way to add payloads to the message is
with the Add Payload service. This service requires the new payload ID
as well as where to get the data from. For instance, a local file. Once
added, the new payload is the current active payload.

![Switch Payload]({{ site.baseurl }}/images/posts/multi-payload/mpm-switch-payload.png)

The third significant component available to work with multi-payload
messages is the For Each conditional, which pretty much works as
expected: it will execute a service (list) for each of the payloads.
There is the option to perform this execution in parallel if so desired.

![For Each]({{ site.baseurl }}/images/posts/multi-payload/mpm-for-each.png)

The final few components that are currently available for handling
multi-payload messages are a MIME encoder/decoder where each payload is
a separate MIME part, and a splitter and aggregator that can be used in
a message splitting service and message joining service respectively.
Used in combination with the Split Join service they are not too
dissimilar from the For Each conditional mentioned above.

End user [documentation][], with example XML configuration, is
available.

[documentation]: https://interlok.adaptris.net/interlok-docs/advanced-multi-payload-messages.html
