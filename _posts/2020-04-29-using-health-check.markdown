---
title:             "Using the health-check management component"
description:       "Wrangling the output from health-check using jq"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            [quotidian-ennui]
excerpt_separator: <!-- more -->
billboard: /billboards/icon-solutions.png
---

3.10.0 introduced a new [health-check](https://github.com/adaptris/interlok-workflow-rest-services) management component which can be used to either report the overall status of all _known adapters_ in that JVM or to report on a specific workflow/channel; reporting on a specific workflow/channel means that you must know what the unique IDs will be. What it doesn't do is to filter by status so you can't get a list of all the channels that aren't started. However, you can still get the output you need if wrangle the JSON output using JQ.

<!-- more -->

First of all, you'll need to install [jq](https://stedolan.github.io/jq/) for your desired platform. You can get it however you normally install packages on your platform (scoop.sh / apt-get / homebrew etc.).

For our purposes, we have an Interlok instance that's been built with the [health-check](https://github.com/adaptris/interlok-workflow-rest-services) management component enabled. We're running the latest SNAPSHOT artefact: 3.10.1 will change the existing top level json object from `java.util.Collection` to `adapters` to avoid excessive information disclosure.


```
managementComponents=jmx:jetty:health-check
rest.health-check.path=/health/*
```

The Interlok instance itself has 3 channels; 2 of which are started; 1 is not. When you use curl against the health check URL then it returns _200 OK_ along with the status of the adapter.

```console
$ curl -si http://localhost:8080/health
HTTP/1.1 200 OK
Content-Type: application/json
Transfer-Encoding: chunked
Server: Jetty(9.4.28.v20200408)

{"adapters":[{"adapter-state":{"id":"MyInterlokInstance","state":"StartedState","channel-states":[{"channel-state":[{"id":"jetty1","state":"StartedState","workflow-states":[{"workflow-state":{"id":"jetty-workflow","state":"StartedState"}}]},{"id":"jetty2(not-started)","state":"ClosedState","workflow-states":[{"workflow-state":{"id":"jetty-workflow","state":"ClosedState"}}]},{"id":"jetty3","state":"StartedState","workflow-states":[{"workflow-state":{"id":"jetty-workflow","state":"StartedState"}}]}]}]}}]}
```

What we really want is to just have a JSON array of channels, with their status. We will arbitrarily filter the array based on the state (using the jq `select` function), so we either print out those that are started, or those that aren't started.

```console
$ curl -s http://localhost:8080/health | jq -c '.["adapters"][0] | .["adapter-state"] | .["channel-states"][] | .["channel-state"][] | select (.state == "StartedState") | { (.id): .state }' | jq -s -c '.'
[{"jetty1":"StartedState"},{"jetty3":"StartedState"}]

$ curl -s http://localhost:8080/health | jq -c '.["adapters"][0] | .["adapter-state"] | .["channel-states"][] | .["channel-state"][] | select (.state != "StartedState") | { (.id): .state }' | jq -s -c '.'
[{"jetty2(not-started)":"ClosedState"}]

```
