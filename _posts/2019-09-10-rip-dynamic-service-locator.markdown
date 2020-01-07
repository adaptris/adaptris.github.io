---
title:             "R.I.P. Dynamic Service Locator"
description:       "Dynamic services have changed; plus ça change, plus c'est la même chose"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            quotidian-ennui
billboard: /billboards/icon-open-source.png
excerpt_separator: <!-- more -->
---

In 3.8.4 we deprecated `dynamic-service-locator`; this decision wasn't taken lightly, but change and evolution is necessary so that you can get where you want to go. Originally `dynamic-service-locator` came about so that people could make changes to configuration independently of restarting. It was only really useful if your possible configurations didn't require new components (e.g. you always dealt in XML, and never used JSON). In fact a lot of the time it was abused by people _logging to the production system and modifying config directly_ which wasn't nice. Even though it's deprecated; changing the service chain that you execute based on various things at runtime is still possible within Interlok.

<!-- more -->

Building up trading relationships and having matching strategies is all fine and dandy apart from 2 things; The notion of a trading relationship isn't that appropriate within the generalised integration context of Interlok, and the matching strategies were largely pointless, since there was only ever one that was usable: `exact-matching-strategy` and both of these caused a level of indirection that wasn't useful. With that in mind, trading relationships have gone away; and for all of these examples, the matching strategy is always precise and exact so that failure to find a service, means that the wrapping service itself fails.

### Shared services with a decision at runtime.

We can define shared services within Interlok configuration; if only there was some way of dynamically deciding on which shared-service to execute at runtime. `dynamic-shared-service` is your friend and the answer to that particular question; configure all the shared services you will need within the _shared-components_ part of your interlok configuration and address them in your workflow using `dynamic-shared-service`. It is something that renders nicely within the UI and doing this if you have a known number of paths and executions is fairly straightfoward.

```xml
<adapter>
  <shared-components>
    <services>
      <service-list>
        <unique-id>PartnerA-PartnerB-MessageType</unique-id>
        <services>
          <log-message-service/>
        </services>
      </service-list>
      <service-list>
        <unique-id>Partner1-Partner2-MessageType</unique-id>
        <services>
          <always-fail-service/>
        </services>
      </service-list>
      <service-list>
        <unique-id>PartnerX-PartnerY-MessageType</unique-id>
        <services>
          <log-message-service/>
        </services>
      </service-list>
    </services>
  </shared-components>
  <channel-list>
     ... skipped for brevity
     <standard-workflow>
       <consumer class="xxx"/>
       <service-collection class="service-list">
        <!--Extract some metadata as 'source' 'destination' 'msgType' via an XPATH / JSONPath, or perhaps it's already populated -->
        <extract-metadata-somehow-service/>
        <dynamic-shared-service>
          <lookup-name>%message{source}-%message{destination}-%message{msgType}</lookup-name>
        </dynamic-shared-service>
       </service-collection>
     </standard-workflow>
  </channel-list>
</adapter>
```

Depending on the metadata values that are associated with message at runtime, you will execute a different service. Since this style of configuration means everything is already pre-defined, that means that for each change you need to redeploy Interlok; our experience is that it's quite rare that you want to change production configuration without having some checks and measures around what you're doing so we quite like this, as it's explicit. Of course we accept that there are some use-cases where a restart is undesirable which brings us to the direct replacement for `dynamic-service-locator`.

### Using dynamic-service-executor

`dynamic-service-executor` has been around for a long time; it was originally designed for a specific use-case where the services that needed to be executed were attached to the incoming message as an additional MIME part (lets ignore the security implications of that statement). We have extended this so that the interface can derive its service configuration from a number of locations. The direct replacement for `dynamic-service-locator` functionality is to use `dynamic-service-from-url` as the service extractor implementation. The configuration for this is simpler than `dynamic-service-locator` since you don't have to bother doing anything around trading relationships and matching strategies; of course the URL can be either file based or remotely addressable via a supported URL

```xml
<!--Extract some metadata as 'source' 'destination' 'msgType' via an XPATH / JSONPath, or perhaps it's already populated -->
<extract-metadata-somehow-service/>
<dynamic-service-executor>
  <service-extractor class="dynamic-service-from-url">
    <url>file:///path/to/store/%message{source}-%message{destination}-%message{msgType}.xml</url>
  </service-extractor>
</dynamic-service-executor>
```

There are other service extractor implementations which makes this much more flexible than `dynamic-service-locator` anyway. Currently the supported extractors are

* `dynamic-mime-service-extractor` which extracts it from the current payload using a MIME part selector (the original use-case).
* `dynamic-service-from-data-input` which extracts from a standard _DataInputParameter_ which means you can extract it from metadata, or the filesystem, which means you can source it from any previously executed service that can modify metadata (like a `jdbc-data-query-service`).
* `dynamic-service-from-url` as per the example which creates the service from the URL in question.
* `dynamic-service-from-cache` which allows you to extract the service from a cache using the configured key. _We don't have an opinion on how you should populating the cache; just that it needs to be populated already_.
* `dynamic-service-from-database`  which allows you configure an arbitrary SQL statement to select the service out of database; of course you are responsible for mitigating against [little bobby tables](https://xkcd.com/327/).
* `dynamic-default-service-extractor` which just uses the current payload as the source of the service; the point of this is debatable, but it is included for completeness

### Conclusion

`dynamic-service-locator` will be removed at some point in the future; you have a number of options as part of your migration path; you should choose the most appropriate so that you aren't left behind.