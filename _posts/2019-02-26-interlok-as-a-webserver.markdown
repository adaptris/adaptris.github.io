---
title:             "Using Interlok to self-host required artefacts"
description:       "Sometimes you just want to be fully self-contained"
published:         true
tags: [interlok, interlok-hammer]
categories: [interlok, interlok-hammer]
author:            quotidian-ennui
billboard: /billboards/icon-orders.png
excerpt_separator: <!-- more -->
---

When you deploy Interlok, there are often a number of supporting items that need to be deployed such as transformation definitions, schemas and the like. They're sometimes developed in conjunction with the solution and live within your source code repository. Often these assets need to include other files; typically a schema might refer to an external DTD to define various elements rather than having it all inline. This situation might cause you issues if your development environment isn't structured the same as your deployment environment.

<!-- more -->

If this was Interlok configuration then you would happily define variables that they would get subsitituted at runtime appropriately; however, that's not possible with the supporting files. One of the techniques that we use is to set up Interlok as a webserver to host these assets; since you have Interlok you have a hammer of infinite possibilities; once again it's ![Interlok Hammer](https://img.shields.io/badge/certified-interlok%20hammer-red.svg) time.

Let's take the situation where you are defining a JSON schema for validating an inbound document. There are some common objects to all the JSON payloads that you'll be handling so you define the schema appropriately to include references :

```
{
  "type": "object",
  "properties": {
    "parameters": {
      "$ref": "file://path/to/parameters.json"
    }
  },
  "required": [
    "parameters"
  ]
}
```

with the supporting file

```
{
  "type": "object",
  "properties": {
    "transactionDate": {
      "type": "string",
      "pattern": "^\\d{4}-\\d{1,2}-\\d{1,2}$"
    }
  },
  "required": [
    "transactionDate"
  ]
}
```

`file://path/to/paramneters.json` could be a URL, but is local since it's only required for this instance; we can avoid any issues around directory structure and the like by making it a localhost URL such as `http://localhost:8080/schemas/parameters.json` and configuring a  workflow to service that URL.

```
<pooling-workflow>
  <consumer class="jetty-message-consumer">
    <unique-id>jetty-consumer</unique-id>
    <destination class="configured-consume-destination">
      <destination>/schemas/*</destination>
    </destination>
  </consumer>
  <service-collection class="service-list">
    <services>
      <read-file-service>
        <unique-id>read-file</unique-id>
        <file-path>${path.on.file.system}/%message{jettyURI}</file-path>
        <content-type-metadata-key>Content-Type</content-type-metadata-key>
        <content-type-probe class="apache-tika-file-probe"/>
      </read-file-service>
      <jetty-response-service>
        <http-status>200</http-status>
        <content-type>%message{Content-Type}</content-type>
      </jetty-response-service>
    </services>
  </service-collection>
</pooling-workflow>
```

`read-file-service` is built into interlok-core.jar; since we have defined a _content-type-metadata-key_ it will attempt to figure out the content of the file and store that as metadata. This means we can use it as the _Content-Type_ header when we serve the file. I'm using `apache-tika-file-probe` (part of interlok-filesystem) instead of the default since `Files#probeContentType()` method is very platform dependent (it seems to always return null on Macs).

Ultimately this means that the deployment is still fully self-contained and you are isolated from any platform differences between development and production.
