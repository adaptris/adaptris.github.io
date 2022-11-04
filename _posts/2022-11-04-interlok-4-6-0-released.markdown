---
title:             "Interlok 4.6.0"
description:       "Interlok 4.6.0 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg9.png
excerpt_separator: <!-- more -->
---

[Interlok 4.6.0](https://development.adaptris.net/installers/Interlok/4.6.0/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/4.6.0/).

<!-- more -->

* Interlok improvements include:

  * [interlok-elastic](https://github.com/adaptris/interlok-elastic) now has an initial implementation of the elastic search 7 SDK named [interlok-elastic-sdk](https://github.com/adaptris/interlok-elastic/tree/develop/interlok-elastic-sdk). This has been introduced as the elastic High Level Rest API is deprecated now in favour of the SDK API.
  * [interlok-azure](https://github.com/adaptris/interlok-azure) has had many improvements; including added support for the search parameter in the O365MailConsumer and improvements to working with attachments from the multi payload message.
  * The UI [Component Search](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-interlok-component-search) has been improved so it doesn't require a connection to a hosted server to work, it has been refactored to use html, js and json instead of an elastic search server.

The formal change log can be found [here](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog)
