---
title:             "Interlok 3.12.0"
description:       "Interlok 3.12.0 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg2.png
excerpt_separator: <!-- more -->
---

[Interlok 3.12.0](https://development.adaptris.net/installers/Interlok/3.12.0/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.12.0/).

<!-- more -->
 
This release is a small release that focuses on some dependency upgrades.

* Interlok Runtime improvements include:
    * Added [STS](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-aws-common/3.12-SNAPSHOT/com/adaptris/aws/STSAssumeroleCredentialsBuilder.html) (Security Token Service) support for building [AWS](https://github.com/adaptris/interlok-aws) (Amazon Web Services) credentials
	* Saxon dependency has been upgraded from [9.9.1-7 to 10.3](https://github.com/adaptris/interlok/pull/561)
	* Added the ability to set [ACL (Access Control List)](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-aws-s3/3.12-SNAPSHOT/com/adaptris/aws/s3/S3ObjectCannedAcl.html) to AWS S3 [upload operations](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-aws-s3/3.12-SNAPSHOT/com/adaptris/aws/s3/UploadOperation.html)
	* The [build-parent](https://github.com/adaptris-labs/interlok-build-parent) can now report on [additional warnings](https://github.com/adaptris-labs/interlok-build-parent/pull/41) from -configcheck

The formal change log can be found [here](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog)
Or you can check the usual [sway presentation](https://sway.office.com/tAeaa6te7XAkp476)
