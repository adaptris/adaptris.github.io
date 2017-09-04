---
layout: post
title: "Interlok as a Reverse Proxy"
date: 2017-09-04 09:00
comments: false
tags: [interlok, heroku]
categories: [interlok, heroku]
published: true
description: "Interlok as a reverse proxy for other Interlok instances; not because we can, but because we might have to"
keywords: "interlok"
---

This is one of those times where having a generic framework is both a blessing and a bit of a curse. One of our customers has a firewall policy that is very strict, only certain ports are open (even internally); the jetty management port had already been opened (8080). We had multiple Interlok instances deployed on the same machine (mainly to partition by logical workunits) and some of those instances were going to expose API endpoints to various other systems. We could have asked for more ports to be open, but there's an overhead and maintenance cost to that.

<!-- more -->

In this particular case, the additional instance workflow was performing a long running task (5-10 minutes). This meant that using the standard Jetty proxy servlet wasn't an option. If we didn't respond back in 60-120 seconds, then the internal gateway (not directly under the customer's control) would respond with a `504 gateway timeout`. If the HTTP client sends `102-Processing` as an _Expect_ header then we will send back a `102` response every 20 seconds or so (as suggested by RFC 2518). However, this caused the Jetty proxy servlet to terminate the socket connection after the first _102_, the client never got the `200 OK` response.

We raised the support ticket, that's the right thing to do; we also configured Interlok that was listening on 8080 as a reverse proxy which allowed us to get some testing done&hellip;

{% highlight xml %}
<pooling-workflow>
  <unique-id>LookupsProxy</unique-id>
  <consumer class="jetty-message-consumer">
    <unique-id>/lookups/*</unique-id>
    <destination class="configured-consume-destination">
      <configured-thread-name>LookupsProxy</configured-thread-name>
      <destination>/lookups/*</destination>
    </destination>
    <parameter-handler class="jetty-http-parameters-as-metadata"/>
    <header-handler class="jetty-http-headers-as-metadata">
      <header-prefix>InboundRequest_</header-prefix>
    </header-handler>
  </consumer>
  <service-collection class="service-list">
    <services>
      <branching-service-collection>
        <unique-id>GenerateURL</unique-id>
        <first-service-id>check-has-query</first-service-id>
        <services>
          <metadata-exists-branching-service>
            <unique-id>check-has-query</unique-id>
            <metadata-key>jettyQueryString</metadata-key>
            <default-service-id>noQuery</default-service-id>
            <metadata-exists-service-id>hasQuery</metadata-exists-service-id>
          </metadata-exists-branching-service>
          <service-list>
            <unique-id>hasQuery</unique-id>
            <services>
              <add-formatted-metadata-service>
                <format-string>http://localhost:8084/%s?%s</format-string>
                <argument-metadata-key>jettyURI</argument-metadata-key>
                <argument-metadata-key>jettyQueryString</argument-metadata-key>
                <metadata-key>proxyURL</metadata-key>
              </add-formatted-metadata-service>
            </services>
          </service-list>
          <service-list>
            <unique-id>noQuery</unique-id>
            <services>
              <add-formatted-metadata-service>
                <format-string>http://localhost:8084/%s</format-string>
                <argument-metadata-key>jettyURI</argument-metadata-key>
                <metadata-key>proxyURL</metadata-key>
              </add-formatted-metadata-service>
            </services>
          </service-list>
        </services>
      </branching-service-collection>
      <standalone-requestor>
        <producer class="apache-http-producer">
          <destination class="metadata-destination">
            <key>proxyURL</key>
          </destination>
          <method-provider class="http-metadata-request-method">
            <metadata-key>httpmethod</metadata-key>
          </method-provider>
          <content-type-provider class="http-metadata-content-type-provider">
            <metadata-key>InboundRequest_Content-Type</metadata-key>
            <default-mime-type>application/json</default-mime-type>
          </content-type-provider>
          <request-header-provider class="apache-http-metadata-request-headers">
            <filter class="composite-metadata-filter">
              <regex-metadata-filter>
                <include-pattern>^InboundRequest_.*</include-pattern>
                <exclude-pattern>^InboundRequest_Host.*</exclude-pattern>
                <exclude-pattern>^InboundRequest_Accept-Encoding.*</exclude-pattern>
                <exclude-pattern>^InboundRequest_Content-Type.*</exclude-pattern>
                <exclude-pattern>^InboundRequest_Content-Length.*</exclude-pattern>
                <exclude-pattern>^InboundRequest_Transfer-Encoding.*</exclude-pattern>
              </regex-metadata-filter>
              <mapped-key-metadata-filter>
                <prefix>^InboundRequest_(.*)</prefix>
                <replacement>$1</replacement>
              </mapped-key-metadata-filter>
            </filter>
          </request-header-provider>
          <response-header-handler class="apache-http-response-headers-as-metadata">
            <metadata-prefix>ProxyResponseHdr_</metadata-prefix>
          </response-header-handler>
          <ignore-server-response-code>true</ignore-server-response-code>
        </producer>
      </standalone-requestor>
      <standalone-producer>
        <unique-id>SendResponse</unique-id>
        <producer class="jetty-standard-response-producer">
          <response-header-provider class="jetty-metadata-response-headers">
            <filter class="composite-metadata-filter">
              <regex-metadata-filter>
                <include-pattern>^ProxyResponseHdr_.*</include-pattern>
                <exclude-pattern>^ProxyResponseHdr_Content-Type.*</exclude-pattern>
                <exclude-pattern>^ProxyResponseHdr_Content-Length.*</exclude-pattern>
                <exclude-pattern>^ProxyResponseHdr_Transfer-Encoding.*</exclude-pattern>
              </regex-metadata-filter>
              <mapped-key-metadata-filter>
                <prefix>^ProxyResponseHdr_(.*)</prefix>
                <replacement>$1</replacement>
              </mapped-key-metadata-filter>
            </filter>
          </response-header-provider>
          <status-provider class="http-metadata-status">
            <code-key>adphttpresponse</code-key>
            <default-status>INTERNAL_ERROR_500</default-status>
          </status-provider>
          <content-type-provider class="http-metadata-content-type-provider">
            <metadata-key>ProxyResponseHdr_Content-Type</metadata-key>
            <default-mime-type>application/json</default-mime-type>
          </content-type-provider>
          <send-payload>true</send-payload>
        </producer>
      </standalone-producer>
    </services>
  </service-collection>

</pooling-workflow>
{% endhighlight %}

The processing sequence is :

1. Receive the request on `/lookups/*`
1. Save all the headers as metadata, prefixed with `InboundRequest_`
  * The keys `httpmethod`, `jettyURI`, `jettyQueryString` are all autopopulated by the jetty consumer.
1. Figure out if we have a query or not via `metadata-exists-branching-service` and generate the correct URL.
1. Issuing the call to the proxied Interlok instance listening on 8084 passing in _most of_ the headers marked as `InboundRequest_*`
  * We used our [apache-http][] optional package because some of the HTTP methods are _PATCH_
  * We filter out some of the headers, because they will be determined by other bits of configuration or aren't needed as they would be defaulted.
  * We use MappedMetadataFilter to strip off the `InboundRequest_` prefix before sending it as an HTTP header
  * We save all the HTTP response headers prefixed with `ProxyResponseHdr_`
  * As we ignore any HTTP error responses from the call; the response code itself is stored against the metadata key `adphttpresponse` automatically
1. Send the stored status code, and headers from the HTTP response back to the client.
  * Again we use MappedMetadataFilter to strip the `ProxyResponseHdr_` prefix.

### Heroku bonus chatter

If you are deploying Interlok instances in Heroku; then Heroku only exposes a single port as the entry point to your instance. If you needed JMX management capabilities __and__ expose an API endpoint, then this kind of proxy pattern will be quite useful as you could enable _ActiveMQ_ as a management component (you need to enable the ActiveMQ HTTP connector); proxy that with a workflow; configure other heroku instances to use `service:jmx:activemq:///http://ui:8080/activemq` as their JMX URL, and use a single UI instance deployed in Heroku to manage other adapter instances.

![HerokuAdapter+ActiveMQ]({{ site.baseurl }}/images/posts/heroku-activemq-proxy.png)

AdapterUI (via the `jetty` management component), ActiveMQ (via the `activemq` management component) and ProxyWorkflow are all running in the same JVM in the same heroku instance. The proxy workflow is configured to forward all requests for _/activemq_ to the HTTP endpoint configured in ActiveMQ. Under the covers, serialisation of the JMX objects is handled by XStream (the default). The only sticking point here is that if you are using the bundled _derby_ database then it can take longer than 30 seconds to startup the UI instance (as the ephemeral heroku filesystem can be quite slow); you will probably need an external database to host the UI settings (or still use derby but the _in-memory_ variant of the database).

[apache-http]: https://development.adaptris.net/nexus/content/groups/public/com/adaptris/adp-apache-http/