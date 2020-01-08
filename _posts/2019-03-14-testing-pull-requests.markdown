---
title:             "Testing Interlok PRs"
description:       "How to test an Interlok Pull Request"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            mcwarman
billboard: /billboards/icon-data-integration.png
excerpt_separator: <!-- more -->
---

There's been a recent change to use pull requests to manage code changes this amongst other things allows for reviews.

One thing that might not be obvious is how to test the PR.<!-- more -->

What we'll make use of is the [interlok-install-builder][] and gradles ability to publish to the local maven repository.

The steps are as follows:

1. Build and Publish Pull Request to local maven repository
1. Build Interlok install

We'll use a real world example: [INTERLOK-2682](https://github.com/adaptris/interlok-apache-http/pull/20).

## Build and Publish Pull Request to local maven repository

The following commands are an example of checking out github project, switching to feature branch and publishing to local maven:

```bash
git clone git@github.com:adaptris/interlok-apache-http.git
cd interlok-apache-http
git checkout feature/INTERLOK-2682
./gradlew clean publishToMavenLocal
```

Now you should have the artefact present in your maven repository:

```bash
ls -lrt ~/.m2/repository/com/adaptris/
total 0
drwxrwxrwx 0 matthew matthew 4096 Mar 14 09:30 interlok-apache-http
```

It's worth noting that if you have other SNAPSHOT artefacts present it would be advisable to remove them as these will be used over others stored on nexus, this should be done prior to `publishToMavenLocal`:

```bash
find ~/.m2/repository/com/adaptris/ -name "3.8-SNAPSHOT" -exec rm -rf {} \;
```


## Build Interlok install

Now that we have a version of code deployed locally we can update our [interlok-install-builder][] to create an Interlok install.

Within the `build.gradle` add  `mavenLocal()` to your repositories, and make sure you have an entry for the built artefact in your dependencies:

```gradle
repositories {
  mavenLocal()
  maven { url "$defaultNexusRepo" }
  maven { url "$nexusBaseUrl/nexus/content/groups/public" }
  //...
}

dependencies {
  //...
  interlokRuntime group: "com.adaptris", name: "interlok-apache-http", version: interlokCoreVersion, changing: true
  //...
}
```

Now you can build and start your adapter (see [interlok-install-builder][] for more details):

```
./gradlew cleanInstall install
cd ./build/install/interlok-install-builder
java -jar ./lib/interlok-boot.jar
```

Now you should have a version of Interlok running with PR changes in the mix.

[interlok-install-builder]: https://github.com/adaptris-labs/interlok-install-builder
