---
title:             "Shared service lists in service tester"
description:       "How to use shared-components in service tester"
published:         true
categories:        [interlok, service-tester]
tags:              [interlok, service-tester]
author:            [mcwarman]
keywords:          "interlok, service-tester, gradle"
excerpt_separator: <!-- more -->
---

Testing shared services is possible in service tester and how to select them is covered in a previous [post]({% post_url 2020-04-16-portable-service-tester-configuration %}). But there may be case when the service that is being tested, contains a call to a shared service.

<!-- more -->

Firstly in order to use any of these you will need to the: Embedded JMX Test client. This is because when the embedded test client is created the shared services will be added to the embedded adapter.

Service tester allows the use shared services in three different ways:

 * Calling the actual service
 * Skipping the execution 
 * Mocking the execution


It should also be noted that at the time of writing these features where not available in the UI.

Within the embedded test client there are two sections;

 * `shared-components` to configure connections or services to be used during the test.

* `shared-components-provider` to select services from the adapter.xml in a similar fashion to service-to-test in a test.

## Calling the actual service

To call the actual service use the `shared-components-provider` to select and therefore create the shared service in your embedded adapter.

The example below will add the `validate-and-extract-document-parameters` to the embedded adapter.

```xml
<test-client class="embedded-jmx-test-client">
  <shared-components-provider>
    <services>
      <service-provider>
        <source class="file-source">
          <file>file:///${service.tester.working.directory}/src/main/interlok/config/adapter.xml</file>
        </source>
        <preprocessors>
          <service-unique-id-preprocessor>
            <service>validate-and-extract-document-parameters</service>
          </service-unique-id-preprocessor>
          <properties-variable-substitution-preprocessor>
            <properties>
              <key-value-pair>
                <key>adapter.json-schema.dir.url</key>
                <value>file:///${service.tester.working.directory}/src/main/interlok/json-schemas</value>
              </key-value-pair>
            </properties>
          </properties-variable-substitution-preprocessor>
        </preprocessors>
      </service-provider>
    </services>
  </shared-components-provider>
</test-client>
```

## Skipping the execution

To skip the execution this is done using the `shared-components` section, and by creating a null-service with the id of the shared service.

The below example will "skip" the execution of `upload-file-to-s3` by instead calling `null-service`.

```xml
<test-client class="embedded-jmx-test-client">
  <shared-components>
    <connections/>
    <services>
      <null-service>
        <unique-id>upload-file-to-s3</unique-id>
      </null-service>
    </services>
  </shared-components>
</test-client>
```

## Mocking the execution

To mock the execution this is also done using the `shared-components` section, by creating a service to do your desired mock.

The below example is mocking a login service list that is designed to return an Authorization metadata key.

```xml
<test-client class="embedded-jmx-test-client">
  <shared-components>
    <connections/>
    <services>
      <add-metadata-service>
        <unique-id>login</unique-id>
        <metadata-element>
          <key>Authorization</key>
          <value>Bearer 1234</value>
        </metadata-element>
      </add-metadata-service>
    </services>
  </shared-components>
</test-client>
```
