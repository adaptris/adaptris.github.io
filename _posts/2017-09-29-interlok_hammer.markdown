---
title:             "Interlok Hammer"
description:       "When all you have is a hammer; everything looks like a nail..."
date:              2017-09-29 08:00:00
categories:        [interlok, interlok-hammer]
tags:              [interlok, interlok-hammer]
author:            mcwarman
---

The term _Interlok Hammer_ came about when our chief architect was describing a client’s use of the Interlok product.<!-- more -->

The use case was an Interlok adapter receiving signed instances of [system-command-executor] service to execute arbitrary system commands to provision processing units.

And then the phrase was born...

Although it started as an internal joke, it has also made me realise a few things. The very nature of product and its ability to chain together arbitrary actions, combined with a person’s pragmatic approach to problem solving, will produce things far from what the product was ever intended to do. Which for me is one of those times I use the word cool (arguably incorrectly)!

More and more examples will pop up from our customers, consultants, or internally (some of which may end up on GitHub or via this blog), there's even a tongue in cheek badge created for projects to wear with pride:

![Interlok Hammer](https://img.shields.io/badge/certified-interlok%20hammer-red.svg)

Some of my favourite Interlok Hammers examples are:

* One that replies to your email with data you requested
* Another that manages your DevOps pipeline for your Interlok config ([Blog post][interlok-devops])
* Another that stores speed test results in elastic search ([GitHub project][interlok-speedtest-elastic])

[system-command-executor]: https://development.adaptris.net/javadocs/latest-stable/Interlok-API/com/adaptris/core/services/system/SystemCommandExecutorService.html
[interlok-devops]: {% post_url 2017-10-02-interlok-cd %}
[interlok-speedtest-elastic]: https://github.com/quotidian-ennui/interlok-speedtest-elastic
