---
title:             "Paged APIs"
description:       "Dealing with Paged APIs in Interlok"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            mcwarman
billboard: /billboards/icon-single-view.png
excerpt_separator: <!-- more -->
---

Integration recently is all about APIs: exposing, consuming, and everything in between. 

One style of API that isn't always obvious on how to deal with is a paged API. These return a fixed number of results and then a number of pages.<!-- more -->

For Example:

```json
GET /api/something?page_size=10&page=1
{
  "data" : [
    //..
  ],
  "pagination": {
    "results": 22,
    "page": 1,
    "page_size": 10,
    "pages": 3
  }
}
```

With the introduction of `interlok-config-conditional` and `interlok-expressions` we can deal with these pages elegantly:

```xml
<service-list>
  <services>
    <add-metadata-service>
      <unique-id>add-paging-metadata</unique-id>
      <metadata-element>
        <key>page_size</key>
        <value>10</value>
      </metadata-element>
      <metadata-element>
        <key>page_number</key>
        <value>1</value>
      </metadata-element>
      <metadata-element>
        <key>pages</key>
        <value>1</value>
      </metadata-element>
    </add-metadata-service>
    <while>
      <unique-id>loop-over-page</unique-id>
      <condition class="expression">
        <algorithm>%message{page_number}&lt;=%message{pages}</algorithm>
      </condition>
      <then>
        <service class="service-list">
          <unique-id>query-and-store-documents</unique-id>
          <services>
            <shared-service>
              <lookup-name>request-and-store</lookup-name>
              <unique-id>request-and-store</unique-id>
            </shared-service>
            <json-path-service>
              <unique-id>get-pages</unique-id>
              <source class="string-payload-data-input-parameter"/>
              <json-path-execution>
                <source class="constant-data-input-parameter">
                  <value>$.pagination.pages</value>
                </source>
                <target class="metadata-data-output-parameter">
                  <metadata-key>pages</metadata-key>
                </target>
              </json-path-execution>
            </json-path-service>
            <expression-service>
              <unique-id>increment-page_number</unique-id>
              <result class="metadata-data-output-parameter">
                <metadata-key>page_number</metadata-key>
              </result>
              <algorithm>$1 + 1</algorithm>
              <parameters>
                <metadata-data-input-parameter>
                  <metadata-key>page_number</metadata-key>
                </metadata-data-input-parameter>
              </parameters>
              <result-formatter class="numerical-result-formatter"/>
            </expression-service>
          </services>
        </service>
      </then>
      <max-loops>0</max-loops>
    </while>
  </services>
</service-list>
```

The `shared-service` `request-and-store` would make a call to the API using the page_number and does something with the payload.

That something could be storing in S3 Bucket, or storing on the filesystem (maybe using the `aggregating-fs-consume-service`) after the loop.