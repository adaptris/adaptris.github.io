---
title:             "Interlok Hammer"
description:       "When all you have is a hammer; everything looks like a nail..."
date:              2017-09-29 08:00:00
categories:        [interlok, interlok-hammer]
tags:              [interlok, interlok-hammer]
billboard: /billboards/icon-solutions.png
author:            mcwarman
---

The term _Interlok Hammer_ came about when our chief architect was describing a consultants use of the Interlok product as a solution for a client<!-- more -->.

The use case being an Interlok adapter receiving signed instances of [system-command-executor] service to dynamically execute arbitrary system commands to provision processing units.

And with that a phrase was born...

It started as an internal joke, but it made me realise something. The combination of the products ability to chain together arbitrary actions and a person’s pragmatic approach to problem solving, will eventually produce things far from what the product was ever intended to do. Which for me is one of those times I use the word cool (arguably incorrectly)!

More and more use cases will pop up from our customers, consultants, and developers (some may end up on GitHub or this blog). There's also a tongue in cheek badge, created for projects to wear with pride:

![Interlok Hammer](https://img.shields.io/badge/certified-interlok%20hammer-red.svg)

Some of my favourite examples of Interlok Hammers are:

* One that replies to your email with data you requested ([Blog post][interlok-mail])
* Another that manages your DevOps pipeline for your Interlok config ([Blog post][interlok-devops])
* Another that stores speed test results in elastic search ([GitHub project][interlok-speedtest-elastic])

Happy Hammering!

[system-command-executor]: https://development.adaptris.net/javadocs/latest-stable/Interlok-API/com/adaptris/core/services/system/SystemCommandExecutorService.html
[interlok-devops]: {% post_url 2017-10-02-interlok-cd %}
[interlok-mail]: {% post_url 2017-10-09-youve_got_mail %}
[interlok-speedtest-elastic]: https://github.com/quotidian-ennui/interlok-speedtest-elastic
