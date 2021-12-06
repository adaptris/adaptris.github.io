---
title:             "Interlok 4.3.0"
description:       "Interlok 4.3.0 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg6.png
excerpt_separator: <!-- more -->
---

[Interlok 4.3.0](https://development.adaptris.net/installers/Interlok/4.3.0/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/4.3.0/).

<!-- more -->

* Interlok improvements include:

  * There is a new '[Interlok Component Search](https://interlok.adaptris.net/interlok-docs/#/pages/overview/adapter-component-search)' feature in our [documentation](https://interlok.adaptris.net/interlok-docs/#/) allowing you to discover all our configurable components
  * A new service collection editor has been added to the [Config Page](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-config-screens?id=navigating-the-config-page) that changes the nested display of services in the list container (needs the [User Preference](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-user-preferences?id=global-preferences) 'enable technical preview features' to be enabled)
  * [UI Templates](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-templates) now display the list of required [optional components](https://interlok.adaptris.net/interlok-docs/#/pages/user-guide/adapter-optional-components) to use the given template
  * New [documentation page](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-auto-register-to-remote) that explains how to auto register an adapter to remote UI Dashboard
  * The [Solace](https://github.com/adaptris/interlok-solace) JCSMP has had many [improvements](https://github.com/adaptris/interlok-solace/pull/24), including updates to use their latest API and add advanced connection/session properties; Reworking of the per-message-properties to be resolvable items, so we can use static and dynamic values; and improvements to the async event handler.
  * A new [command line installer](https://development.adaptris.net/installers/early_access/4.3.0B1/interlok-installer-cmd.tar.gz) is available to [download](https://development.adaptris.net/installers/early_access/4.3.0B1/)
  * New ['proxy' management component](https://github.com/adaptris/interlok-workflow-rest-services/tree/develop/interlok-proxy) that allows you, for example, to have an API endpoint serviced by Interlok and have Jolokia enabled

The formal change log can be found [here](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog)
Or you can check the usual [sway presentation](https://sway.office.com/ai6mIvyBFGdNTln3)
