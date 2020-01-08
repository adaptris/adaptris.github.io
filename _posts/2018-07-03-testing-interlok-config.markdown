---
title:             "Testing Interlok config"
description:       "Integration isn't hard so why should testing it be?!"
published:         true
categories:        [interlok, service-tester]
tags:              [interlok, service-tester]
author:            mcwarman
billboard: /billboards/icon-big-data.png
excerpt_separator: <!-- more -->
---

I've been involved in many integration projects sometimes you'll hear a developer say "You can test it when its deployed", often true (I've caught myself saying it) but that thinking only applies to integration testing what about the world of unit testing.

<!-- more -->

# Setting the Scene

Consider the following configuration fairly simple as far a configuration goes, but it fits the argument that it's only testable in an integration environment because of the required database connectivity and a running adapter with embedded jetty.

![testing config]({{ site.baseurl }}/images/posts/testing-config.png)

But the parts that need most testing (or could go most wrong), is the routing and the mapping. Neither of which need connectivity to be tested in isolation.

The Interlok UI does provide the ability to test steps which is useful when developing, but is not easily repeatable once you've moved on.

Something that has been quietly available for a while is the `interlok-service-tester`, one limitation this had was the requirement to write the config in XML which isn't very "cool" at all. As of 3.7.2 the service-tester can now be configured using the GUI (option available under Config dropdown).

This enables two things: One is it speeds up the ability to deliver tests, and two is after some initial setup testing can be handed over to other teams who maybe aren't as comfortable with the XML bed we've made for our selves.

# Testable Code

Firstly as a developer you need to make your Interlok code testable, in terms of Interlok it means the same as it does in any other programming language, making sure your code is written with testing in mind.

This can be achieved my wrapping services that we want to test as a group of services in a `service-list`, consider the above example to make it more testable we could do the following:

![testing config revisited]({{ site.baseurl }}/images/posts/testing-config-revisited.png)

Now we have a testable `service-list` (conveniently) called mapping.

# Service Tester

In order to use service-tester you need to make sure the jar is in your lib directory if you're still copying jars from optionals then it should be there if you're managing your dependency else where add it in there.

Service-tester is split into three parts:

* Test Lists which contain tests
* Tests which you configure the services to be tested, and contain test cases
* Test Cases where input messages and corresponding assertions are configured

## Configuring Tests

The first thing you'll need to do when using service-tester is configure your test (Technically it would be a `test-list` but the GUI gives you that for free). The test defines the `service-to-test` this is made up of two parts: Source and Preprocessors.

Source will more often than not be a file, where as preprocessors will fit the needs of what you need them to do, preprocessors are the bits that you set what to extract and configure in your config.

![service-tester test]({{ site.baseurl }}/images/posts/testing-service-tester-test.png)

The above example is using the config stored in `/opt/interlok/config/adapter.xml` and extracting the `mapping` service as the `service-to-test` using `service-unique-id-preprocessor`.

You can run the config here now to see if you're `service-to-test` gets extracted, it won't do anything useful at this point though.

_TIP: Other preprocessors include variable substitution and xpath extracts_

## Test Cases - Input Messages and Assertions

Now we've successfully extracted the "unit" we want to test.  We can test it by adding a test case. Test cases contain two parts: message-providers and assertions.

![service-tester test case]({{ site.baseurl }}/images/posts/testing-service-tester-test-case.png)

The above configuration has an input payload (which is replicating what would be returned by the `jdbc-query-serivce`) and assertion that checks the xpath value in returned response.

_TIP: A useful way to check the results of a returned query is to use the `always-fail` assertion as it will display the returned message as a part of the error._

## Running

Now we can execute our test and we can see whether it worked as expected:

**Success**

![service-tester success]({{ site.baseurl }}/images/posts/testing-service-tester-success.png)

**Failure**

![service-tester success]({{ site.baseurl }}/images/posts/testing-service-tester-failure.png)

# Conclusion

A working example is available in a [github project](https://github.com/adaptris/interlok-service-tester-example).

This blog post has attempted to scratch the surface on what service-tester has to offer, hopefully you've made it this far and feel a little more educated on the matter.

Once you've created your config you could always consider integrating it as a part of continuous integration pipeline something we hope to cover in future post.

See the [javadocs](https://development.adaptris.net/javadocs/latest-stable/optional/service-tester/) and [documentation](http://interlok.adaptris.net/interlok-docs/service-tester-introduction.html) for additional information.
