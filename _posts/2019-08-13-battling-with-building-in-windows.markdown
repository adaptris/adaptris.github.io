---
title:             "Battling with builds in Windows"
description:       "Getting gradle builds in windows working."
published:         true
categories:        [interlok]
tags:              [interlok]
author:            mcwarman
excerpt_separator: <!-- more -->
---

More and more we are using build tools to assist in the creation of Interlok artefacts, until recently there were a few scenarios that didn't work in windows this meant we had to use WSL to run our builds.

<!-- more -->

The reasons for the build failures were unrelated to anything we'd done but related to the way gradle builds its classpath and limitation in windows:

```
Caused by: java.io.IOException: Cannot run program "...\java.exe" (in directory "...\build"): CreateProcess error=206, The filename or extension is too long
```

In order to fix this, we created a temporary jar that contained the classpath, this manifests itself as (pun intended):

```gradle
task tempLauncherJar(type: Jar) {
    appendix = "launcher"
    manifest {
        attributes ("Class-Path": configurations.runtime.collect { "file:///" + it.getCanonicalPath() }.join(' '))
    }
}

task interlokVersion(type: JavaExec, dependsOn: processConfig) {
    dependsOn tempLauncherJar
    workingDir = new File("${buildDir}")
    classpath = files(tempLauncherJar.archivePath)
    main = 'com.adaptris.core.management.SimpleBootstrap'
    args "-version"
}
```

