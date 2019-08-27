---
title:             "Service testing my mappings"
description:       "How can I be confident that my upgrade works?"
published:         true
categories:        [interlok, service-tester]
tags:              [interlok, service-tester]
author:            jameswickham
excerpt_separator: <!-- more -->
---

Recently I was involved in a task to test any possibly impacts updating the version of Saxon could have on the Interlok’s xslt transformation service and it got me thinking that the service tester could also be very useful when updating any mappings from xslt 1.0 to 2.0 to make sure we still get the same output.

<!-- more -->

This blog is written with the assumption that you have the service tester jar already included in your lib directory.


## Add some additional jars to the lib directory

I just need to add a couple of extra jars inside my interlok installation lib directory e.g. C:\adaptris\interlok\lib.

* [xmlunit-1.2.0.jar](https://repo.spring.io/plugins-release/org/custommonkey/xmlunit/com.springsource.org.custommonkey.xmlunit/1.2.0/com.springsource.org.custommonkey.xmlunit-1.2.0.jar)
* [junit-3.8.2.jar](https://repo.spring.io/libs-release/org/junit/com.springsource.junit/3.8.2/com.springsource.junit-3.8.2.jar)

## The Interlok UI Service Tester

So now I have the correct jars installed I can go ahead and start up my adapter and open up the service tester from the config drop down list:

![service-tester-open]({{ site.baseurl }}/images/posts/test-mapping/Es5SipC.png)

Once I’ve opened the service tester I want to select “Add Test” which will then bring up a window where I can setup my test.

![service-tester-add]({{ site.baseurl }}/images/posts/test-mapping/add-test.png)

I’ve given my test a Unique ID which is relevant to the type of test I will be conducting. I have also included the service I want to test, in this case I’m testing the xml transformation service.
In this case I don’t need any pre-processors so set I’ve it to null. 

<!-- more -->

Now I have our test setup I need to add a test case. So first of all select “Add Test Case” which will bring up a window where I can configure my test case.

![service-tester-add-one]({{ site.baseurl }}/images/posts/test-mapping/test-case-1.png)

So I need to set a relevant unique ID for the test case too.

<!-- more -->

I have also included inline the input payload I want to test with, I could also point the service tester to a file on my local system.

<!-- more -->

Now the Assertion I want to make is the output file after running it through the mapping matches the xml payload I have provided inline.

<!-- more -->

Once I have set this up I can save it. Now I have my test setup I can run it. This first test I am expecting to pass as we have not yet altered the mapping version so the output should be exactly the same.

![service-tester-result]({{ site.baseurl }}/images/posts/test-mapping/result.png)

I ran the test and as expected it passed! Brilliant as this establishes the test control is correct.

<!-- more -->

Now I will go into the mapping and change the version to xslt 2.0, I can do this by changing it at the top of the xslt stylesheet.

![service-tester-mapping-change]({{ site.baseurl }}/images/posts/test-mapping/mapping-change.png)

Ok now I’ve changed the version I can run the test again.

![service-tester-result2]({{ site.baseurl }}/images/posts/test-mapping/result2.png)

Ok so this time the test’s assertion has failed, I can now hover over the test result and it gives me the details as to why it failed.

<!-- more -->

So in this case it’s because the &lt;result&gt; field returned multiple values within in which is a result of the way xslt 2.0 handles xsl:value-of. 
In xslt 2.0 it returns all occurrences of an xpath whereas xslt 1.0 returns only the first occurrence.

# Conclusion

Because of running this test I have identified that my mapping will need to be edited to accommodate updating the xslt version.
