---
title:             "Interlok 3.11.0"
description:       "Interlok 3.11.0 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg12.png
excerpt_separator: <!-- more -->
---

[Interlok 3.11.0](https://development.adaptris.net/installers/Interlok/3.11.0/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.11.0/).

<!-- more -->
 
* The UI Config Page has improved support for project documentation (README.md & markdown comments in channels, workflows and collections)
* Interlok Runtime improvements include:
    * [ProduceDestination and ConsumeDestination have been deprecated](https://github.com/adaptris/interlok/blob/develop/docs/adr/0005-remove-produce-destination.md) and replaced with simplified configuration elements
    * [Json](https://github.com/adaptris/interlok-json) [Schema Validation](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-json/3.11-SNAPSHOT/com/adaptris/core/json/schema/JsonSchemaService.html) now reports more detailed exceptions
    * [JsonTransformService](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-json/3.11-SNAPSHOT/com/adaptris/core/transform/json/JsonTransformService.html) now supports json and yaml
    * [JSON](https://github.com/adaptris/interlok-json) can now be [encoded](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-json-web-token/3.11-SNAPSHOT/com/adaptris/core/jwt/JWTEncoder.html) and [decoded](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-json-web-token/3.11-SNAPSHOT/com/adaptris/core/jwt/JWTDecoder.html) to the [JWT](http://www.jsonwebtoken.io) ([JSON Web Token](https://github.com/adaptris/interlok-json-web-token)) standards
    * Added support for a [HTTP endpoint](https://github.com/adaptris/interlok-workflow-rest-services) that [Prometheus can scrape for metrics](https://interlok.adaptris.net/interlok-docs/#/pages/advanced/advanced-profiler-prometheus)
    * Integrated asynchronous publishing via the JCSMP API into the Interlok bridging model.  We now support the basic message payload types of text, bytes, xml-bytes and xml-content.

The formal change log can be found [here](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog)
Or you can check the usual [sway presentation](https://sway.office.com/egFRVY8G8BrHzfv4)

# Screenshots

new 'README.md' support  :
![readme 01]({{ site.baseurl }}/images/posts/\release-3110/readme-1.png)
![readme 02]({{ site.baseurl }}/images/posts/\release-3110/readme-2.png)
![readme 03]({{ site.baseurl }}/images/posts/\release-3110/readme-3.png)
![readme 04]({{ site.baseurl }}/images/posts/\release-3110/readme-4.png)

markdown comments  :
![comments 01]({{ site.baseurl }}/images/posts/\release-3110/comments-1.png)
![comments 02]({{ site.baseurl }}/images/posts/\release-3110/comments-2.png)
![comments 03]({{ site.baseurl }}/images/posts/\release-3110/comments-3.png)
![comments 04]({{ site.baseurl }}/images/posts/\release-3110/comments-4.png)