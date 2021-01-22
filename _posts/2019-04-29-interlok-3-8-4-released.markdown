---
title:             "Interlok 3.8.4"
description:       "Interlok 3.8.4 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg4.png
excerpt_separator: <!-- more -->
---

[Interlok 3.8.4](https://development.adaptris.net/installers/Interlok/3.8.4/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.8.4/).

<!-- more -->

The key highlights are :

* [Config projects](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-config-project) continued improvements: better UX for loading projects; improved variable usage when moving/coping components with existing variables; and various improvements around the importing existing config with multiple variable sets; and configurability of the x-includes root location.
* The [Component Search](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-interlok-component-search) has been improved so search results now link to the optional component page and vice versa.
* The [DynamicServiceExecutor](https://development.adaptris.net/javadocs/latest-stable/Interlok-API/com/adaptris/core/services/dynamic/DynamicServiceExecutor.html) has been enhanced and can be used as a simplified [DynamicServiceLocator](https://development.adaptris.net/javadocs/latest-stable/Interlok-API/com/adaptris/core/services/dynamic/DynamicServiceLocator.html) (which has been deprecated).
* OAuth components have been improved to support the generation of the OAuth Signature for OAUTH1.0 / RFC 5849 (Optional component: interlok-oauth-generic)
* A new service-list implementation that auto maps against StaX implementations (Optional component: Interlok-stax)
* New [XML Exception Report service](https://development.adaptris.net/javadocs/latest-stable/Interlok-API/com/adaptris/core/services/exception/XmlExceptionReport.html) that includes workflow ID and Message

The formal [change log](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog) can be found [here](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog). 
Or you can check the usual [sway presentation](https://sway.office.com/53JmWZDwxTJlsU46)


# Screenshots

Config projects - load local project from filesystem:
![xpath validation for variables]({{ site.baseurl }}/images/posts/release-384/history1.png)

![xpath validation for variables]({{ site.baseurl }}/images/posts/release-384/history2.png)

![xpath validation for variables]({{ site.baseurl }}/images/posts/release-384/history3.png)

Config projects - x-incs base folder config:
![xpath validation for variables]({{ site.baseurl }}/images/posts/release-384/xincs.png)

Component Search - examples:
![xpath validation for variables]({{ site.baseurl }}/images/posts/release-384/search-link1.png)

![xpath validation for variables]({{ site.baseurl }}/images/posts/release-384/search-link2.png)
