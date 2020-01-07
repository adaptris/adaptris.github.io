---
title:             "Abusing Interlok as a programming language"
description:       "Interlok configuration probably isn't Turing complete; but we can write code in XML..."
published:         true
categories:        [interlok]
tags:              [interlok]
author:            [quotidian-ennui]
billboard: /billboards/icon-inventory.png
excerpt_separator: <!-- more -->
---

We're busy migrating away from Bitbucket; not because there's anything wrong with it; but because as an organisation that's grown out of 6+ companies, you have to rationalise and standardise things like source control; we're using GitHub, Bitbucket, multiple on-premise Gitlab instances, Azure TFS; you get the picture. Apparently we need a spreadsheet (it's always a spreadsheet) of all the projects that are currently in Bitbucket and when they were last used. That's right; it's ![Interlok Hammer](https://img.shields.io/badge/certified-interlok%20hammer-red.svg) time.

<!-- more -->

It would have been easy to go through and just export the list of projects in Bitbucket since you can already order by last updated date; but I wanted to include the repository type information (i.e. Mercurial or Git) since this also has a tangential impact on the migration since there might be some extra steps in the migration path to migrate from mercurial. How then to generate the CSV for the manager types.

We're going to use the bitbucket REST API and iterate over list of projects and generate a CSV from there. The interesting point here is not that I chose Interlok to do this, but that the time it took _was comparable to doing it manually_; and the output of the project can be used to change the output if requirements change. I could have scripted it, used a formal programming language, but I'm not sure that it would have been any quicker to completion than the solution that I created.

The sequence of steps can be described pretty easily using some kind of pseudo-code:

```
REPEAT
  GET url (https://api.bitbucket.org/repositories/adaptris)
  FOR EACH repo
  DO
    APPEND CSV record to file.
  DONE
  IF has-next-page; url = next-page
UNTIL url == "";
```

## Interlok Design

We can model that very easily in Interlok; I opted to make the trigger an HTTP Request which returns CSV data as the HTTP response; so GET returning `text/plain` is our baseline behaviour. Bearing in mind some of the limitations of Interlok; we currently don't have the capability to support multiple payloads (this will come); so we have to use a staging file to store the data while we iterate over the URLs.

Our Interlok sequence can be elaborated to be:

```
Generate a filename for storing the CSV "csvFilename"
Generate the initial URL and store it as metadata (nextPage)
Write the CSV header that you want to "csvFilename"
DO
  HTTP GET using nextPage as the URL.
  Remove "nextPage" from metadata.
  Check if there's a next page; store it as "nextPage"
  SPLIT/JOIN on $.values {
    MAP object into a simplified format
  }; JOIN it as a JSON Array
  JSON Array To CSV
  APPEND it to "csvFilename"
WHILE "nextPage" != null
READ "csvFilename"
Return the HTTP response
```

We can split this out into a number of conceptual steps:

## Pre-loop activities

* Generate the temporary CSV filename.
* Write the CSV header (after generating as a payload) to the temporary filename.
* Generate the initial URL.

```
<generate-unique-metadata-value-service>
  <metadata-key>csvFilename</metadata-key>
  <generator class="guid-generator"/>
</generate-unique-metadata-value-service>
<payload-from-metadata-service>
  <template><![CDATA[repository_name,repository_type,project,last_updated
]]></template>
</payload-from-metadata-service>
<standalone-producer>
  <producer class="fs-producer">
    <destination class="configured-destination">
      <destination>file:///./fs/</destination>
    </destination>
    <create-dirs>true</create-dirs>
    <fs-worker class="fs-overwrite-file"/>
    <filename-creator class="metadata-file-name-creator">
      <metadata-key>csvFilename</metadata-key>
    </filename-creator>
  </producer>
</standalone-producer>
<add-metadata-service>
  <metadata-element>
    <key>nextPage</key>
    <value>https://api.bitbucket.org/2.0/repositories/adaptris</value>
  </metadata-element>
</add-metadata-service>
```

## Iterating over the URL

It's a `do-while`; with a `max-loops=-1`. Inside the loop we

* Query the URL (`apache-http-request-service`).
* Remove the "next-page" metadata key (`metadata-filter-service`).
* Re-create the "next-page" metadata key if it's possible (`json-path-service`).
* Split/Join to process each of the JSON values.
* JSON array to CSV, omitting the headers (since we have already written it).

Note that I haven't bothered with an OAUTH workflow here, I'm just using my username, and a _Bitbucket App Password_.

```
<do-while>
  <condition class="not">
    <condition class="metadata">
       <operator class="is-empty"/>
       <metadata-key>nextPage</metadata-key>
    </condition>
  </condition>
  <then>
    <service class="service-list">
      <unique-id>bitbucket-repos-to-csv</unique-id>
      <services>
        <apache-http-request-service>
          <unique-id>list-repos</unique-id>
          <url>%message{nextPage}</url>
          <method>GET</method>
          <authenticator class="apache-http-dynamic-authorization-header">
            <username>${bitbucket.api.user}</username>
            <password>${bitbucket.api.password}</password>
          </authenticator>
        </apache-http-request-service>
        <metadata-filter-service>
          <unique-id>remove-nextPage</unique-id>
          <filter class="regex-metadata-filter">
            <exclude-pattern>^nextPage.*</exclude-pattern>
          </filter>
        </metadata-filter-service>
        <json-path-service>
          <unique-id>get-next-page</unique-id>
          <source class="string-payload-data-input-parameter"/>
          <json-path-execution>
            <source class="constant-data-input-parameter">
              <value>$.next</value>
            </source>
            <target class="metadata-data-output-parameter">
              <metadata-key>nextPage</metadata-key>
            </target>
            <suppress-path-not-found>true</suppress-path-not-found>
          </json-path-execution>
        </json-path-service>
        <pooling-split-join-service>
          <!-- discussed below -->
        </pooling-split-join-service>
        <json-to-csv>
          <preference-builder class="csv-basic-preference-builder">
            <style>STANDARD_PREFERENCE</style>
          </preference-builder>
          <include-header>false</include-header>
        </json-to-csv>
        <standalone-producer>
          <unique-id>append-to-known-filename</unique-id>
          <producer class="fs-producer">
            <destination class="configured-destination">
              <destination>file:///./fs/</destination>
            </destination>
            <create-dirs>true</create-dirs>
            <fs-worker class="fs-append-file"/>
              <filename-creator class="metadata-file-name-creator">
                <metadata-key>csvFilename</metadata-key>
               </filename-creator>
            </producer>
        </standalone-producer>
      </services>
    </service>
  </then>
  <max-loops>-1</max-loops>
</do-while>
```

## Pooling splitjoin

* Split on a JSON path
* For each of the JSON objects, map it into a simpler one (`json-transform-service`)
  * It's a very simple JOLT mapping.
* Join each of simplified JSON objects into a JSON array.

```
<pooling-split-join-service>
  <unique-id>split-on-values</unique-id>
  <service class="service-list">
    <unique-id>handle-individual-json</unique-id>
      <services>
        <json-transform-service>
          <unique-id>desperate-shaw</unique-id>
          <source-json class="string-payload-data-input-parameter"/>
          <mapping-spec class="constant-data-input-parameter">
            <value>[
  {
    "operation": "shift",
    "spec": {
      "name": "repository_name",
      "scm": "repository_type",
      "project": {
        "name": "project"
      },
      "updated_on": "last_updated"
    }
  }
]</value>
          </mapping-spec>
          <target-json class="string-payload-data-output-parameter"/>
          <metadata-filter class="remove-all-metadata-filter"/>
        </json-transform-service>
      </services>
  </service>
  <splitter class="json-path-splitter">
    <copy-metadata>true</copy-metadata>
    <copy-object-metadata>true</copy-object-metadata>
    <json-source class="string-payload-data-input-parameter"/>
    <json-path class="constant-data-input-parameter">
      <value>$.values</value>
    </json-path>
    <message-splitter class="json-array-splitter"/>
  </splitter>
  <aggregator class="json-array-aggregator"/>
</pooling-split-join-service>
```

## Conclusion

The CSV looks something like:

```
repository_name,repository_type,project,last_updated
ProjectA,hg,Legacy,2017-12-22T01:13:45.232962+00:00
AwesomeWebPortal,hg,Portals,2017-09-11T10:33:14.080179+00:00
interlok-pipeline,git,Interlok,2019-09-04T14:00:13.655863+00:00
...
```

_All in all this took me about 45 minutes_; this isn't significantly longer than how long it would have taken someone to do it manually. We can argue the point about whether it would have been quicker to do it in python/node/go/rust whatever. I'm a developer, and I'm choosing not to write code, in fact, you might think of this as using Interlok as a _drag and drop programming language_.

We even have the capability to modify the JOLT transform so that we expose more information in the CSV file. You could equally point this at other source control systems that have a REST API with minor changes. It just goes to show that you can deliver value quickly with Interlok.

