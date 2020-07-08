---
title:             "Interlok 3.10.2"
description:       "Interlok 3.10.2 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg11.png
excerpt_separator: <!-- more -->
---

[Interlok 3.10.2](https://development.adaptris.net/installers/Interlok/3.10.2/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.10.2/).

<!-- more -->

* [UI Config Projects](https://interlok.adaptris.net/interlok-docs/ui-config-project.html) new ['Optional Components' tab](https://interlok.adaptris.net/interlok-docs/ui-config-project.html#optional-components-tab) displays which additional optional components are needed in the Config XML
* Interlok Runtime improvements include:
    * The [XML schema validation](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.10-SNAPSHOT/com/adaptris/core/transform/XmlSchemaValidator.html) has been improved to report on all schema violations rather than just the first one
    * New endpoints that supply 'readiness' and 'liveness' information in the [interlok-workflow-rest-services](https://github.com/adaptris/interlok-workflow-rest-services) [health-check component](https://github.com/adaptris/interlok-workflow-rest-services#health-check)
    * A new [interlok-aws-s3](https://github.com/adaptris/interlok-aws/tree/develop/interlok-aws-s3) [amazon-s3-extended-copy operation](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-aws-s3/3.10.2B1-RELEASE/com/adaptris/aws/s3/ExtendedCopyOperation.html) which mirrors CopyOperation but exposes additional object tags and object metadata
    * [interlok-jclouds-blobstore](https://github.com/adaptris/interlok-jclouds/tree/develop/interlok-jclouds-blobstore) [jclouds-blobstore-connection](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-jclouds-blobstore/3.10.2B1-RELEASE/com/adaptris/jclouds/blobstore/BlobStoreConnection.html) now supports AWS session tokens when connecting to S3
    * [interlok-aws-common](https://github.com/adaptris/interlok-aws/tree/develop/interlok-aws-common) added support for [Signing HTTP Requests](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-aws-common/3.10.2B1-RELEASE/com/adaptris/aws/apache/interceptor/ApacheSigningInterceptor.html) to the AWS managed Elasticsearch Service
    * Improved performance for BlobStorage object listings, which are now lazy loaded. This covers many components such as aws [ListOperation](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-aws-s3/3.10.2B1-RELEASE/com/adaptris/aws/s3/ListOperation.html), [JsonBlobListRenderer](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-json/3.10.2B1-RELEASE/com/adaptris/core/json/JsonBlobListRenderer.html), [CsvBlobListRenderer](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-csv/3.10.2B1-RELEASE/com/adaptris/csv/CsvBlobListRenderer.html) etc.
    * If you enable the [jetty management component](https://interlok.adaptris.net/interlok-docs/adapter-bootstrap.html#jetty-component) w/o an accompanying jetty.xml configuration, a fail-safe one will be used
    * The docker base images will now use a JDK and not a JRE (as certain things like [CRaSH](https://interlok.adaptris.net/interlok-docs/advanced-shell.html) requires a JDK)

The formal change log can be found [here](https://interlok.adaptris.net/interlok-docs/changelog.html)
Or you can check the usual [sway presentation](https://sway.office.com/s0AcmVBHGKyq0xmf)

# Screenshots

new 'Optional Components' tab  :
![Optional 01]({{ site.baseurl }}/images/posts/\release-3102/optional-tab-1.png)
![Optional 02]({{ site.baseurl }}/images/posts/\release-3102/optional-tab-2.png)
