---
title:             "You've got mail!"
description:       "Interlok providing real time data exports using email as the provider"
date:              2017-10-09 08:00:00
categories:        [interlok, interlok-hammer]
tags:              [interlok, interlok-hammer]
author:            mcwarman
---

Many times working in an IT team you'll be asked to provide an extract of data from _System A_ in excel friendly format. I've seen this solved in many different ways<!-- more -->, a couple being:

* Giving read only database accounts out to the business
* Using Excel VB macros with some embedded database connections
* Developing some sort of bespoke front end button to do it

Each introducing their own management headache:

* Exposing some part of the data you weren't planning to
* Difficulty updating and maintaining versions (everyone loves there local copy)
* Added overhead of maintaining the bespoke front end service

How Interlok has been used to solve this is using a tool that we all use daily... email.

## Solution

Simply the solution has three main parts:

* Listen for a new email in specific mailbox (maybe with a certain subject)
* Get and transform the data
* Send an email back

Technically what this means is:

* Consumer: `raw-mail-consumer` with a `filter-expression`
* Services: `jdbc-data-query-service` with `xml-transformation-service` (just one example of the many service combination options)
* Producer: `multi-attachment-smtp-producer` using `metadata-destination`

All together it looks like this:

<figure class="highlight">
  <figcaption class="g code-caption">adapter.xml</figcaption>
{% highlight xml %}
<adapter>
  <unique-id>mail-to-mail</unique-id>
  <shared-components>
    <!-- not important -->
  </shared-components>
  <channel-list>
    <channel>
      <unique-id>channel</unique-id>
      <consume-connection class="null-connection"/>
      <produce-connection class="null-connection" />
      <workflow-list>
        <standard-workflow>
          <unique-id>workflow</unique-id>
          <consumer class="raw-mail-consumer">
            <destination class="configured-consume-destination">
              <destination>imap://interlok:password@mail:3143/INBOX</destination>
              <filter-expression>SUBJECT=[Rr]eport</filter-expression>
            </destination>
            <poller class="fixed-interval-poller">
              <poll-interval>
                <unit>SECONDS</unit>
                <interval>10</interval>
              </poll-interval>
            </poller>
            <header-handler class="mail-headers-as-metadata">
              <header-filter class="regex-metadata-filter">
                <include-pattern>From</include-pattern>
                <include-pattern>Subject</include-pattern>
              </header-filter>
            </header-handler>
            <delete-on-receive>true</delete-on-receive>
          </consumer>
          <service-collection class="service-list">
            <services>
              <!-- Data extractions and transformation -->
              <add-formatted-metadata-service>
                <format-string>RE: %s</format-string>
                <argument-metadata-key>Subject</argument-metadata-key>
                <metadata-key>emailsubject</metadata-key>
              </add-formatted-metadata-service>
            </services>
          </service-collection>
          <producer class="multi-attachment-smtp-producer">
            <username>interlok</username>
            <password>password</password>
            <destination class="metadata-destination">
              <key>From</key>
            </destination>
            <smtp-url>smtp://mail:3025</smtp-url>
            <mail-creator class="mail-xml-content-creator">
              <attachment-handler class="mail-xml-attachment-handler">
                <xpath>/document/attachment</xpath>
                <filename-xpath>@filename</filename-xpath>
                <encoding-xpath>@encoding</encoding-xpath>
              </attachment-handler>
              <body-handler class="mail-xml-body-handler">
                <xpath>/document/content</xpath>
                <content-type>text/plain</content-type>
              </body-handler>
            </mail-creator>
          </producer>
        </standard-workflow>
      </workflow-list>
    </channel>
  </channel-list>
</adapter>
{% endhighlight %}
</figure>

A working example is available in a [github project](https://github.com/adaptris-labs/interlok-mail-to-mail).

## Summary

A fairly simple solution that uses an existing technology we are all familiar with, with the luxury of no additional front ends and keeping the configuration in a central location.

Also a: ![Interlok Hammer](https://img.shields.io/badge/certified-interlok%20hammer-red.svg).

## Something Extra

If you're a user of the GUI, you could add the following template file into `./ui-resources/config-templates/workflows/`, and add a new workflow using the `You've Got Mail!` template:

<figure class="highlight">
  <figcaption class="g code-caption">./ui-resources/config-templates/workflows/youve_got_mail.xml</figcaption>
{% highlight xml %}
<standard-workflow
         tmpl-alias="Standard Workflow"
         tmpl-alias-value="standard-workflow"
         tmpl-author="Adaptris"
         tmpl-class-name="com.adaptris.core.StandardWorkflow"
         tmpl-created="2017-10-09T08:00:00"
         tmpl-desc="Reads and Replies to emails"
         tmpl-keywords="mail"
         tmpl-name="You've got mail!"
         tmpl-target-version="3.6.4-RELEASE"
         tmpl-title="You've got mail!"
         tmpl-type="workflow" tmpl-wizard="true"
         wizard-mail-order="0" wizard-mail-desc="Configure the mail connectivity"
>
  <consumer class="raw-mail-consumer">
    <destination class="configured-consume-destination">
      <destination wizard-key="imap-pop3-destination" wizard-desc="IMAP/POP3 Destination" wizard-step="mail" wizard-order="0" wizard-type="string"/>
      <filter-expression wizard-key="imap-pop3-filter" wizard-desc="IMAP/POP3 Filter" wizard-step="mail" wizard-order="1" wizard-type="string">SUBJECT=[Rr]eport</filter-expression>
    </destination>
    <poller class="fixed-interval-poller">
      <poll-interval>
        <unit>SECONDS</unit>
        <interval>10</interval>
      </poll-interval>
    </poller>
    <header-handler class="mail-headers-as-metadata">
      <header-filter class="regex-metadata-filter">
        <include-pattern>From</include-pattern>
        <include-pattern>Subject</include-pattern>
      </header-filter>
    </header-handler>
    <delete-on-receive>true</delete-on-receive>
  </consumer>
  <service-collection class="service-list">
    <services>
      <!-- Data extractions and transformation -->
      <add-formatted-metadata-service>
        <format-string>RE: %s</format-string>
        <argument-metadata-key>Subject</argument-metadata-key>
        <metadata-key>emailsubject</metadata-key>
      </add-formatted-metadata-service>
    </services>
  </service-collection>
  <producer class="multi-attachment-smtp-producer">
    <username wizard-key="smtp-username" wizard-desc="SMTP Username" wizard-step="mail" wizard-order="3" wizard-type="string"/>
    <password wizard-key="smtp-password" wizard-desc="SMTP Password" wizard-step="mail" wizard-order="4" wizard-type="password"/>
    <destination class="metadata-destination">
      <key>From</key>
    </destination>
    <smtp-url wizard-key="smtp-url" wizard-desc="SMTP URL" wizard-step="mail" wizard-order="2" wizard-type="string"/>
    <mail-creator class="mail-xml-content-creator">
      <attachment-handler class="mail-xml-attachment-handler">
        <xpath>/document/attachment</xpath>
        <filename-xpath>@filename</filename-xpath>
        <encoding-xpath>@encoding</encoding-xpath>
      </attachment-handler>
      <body-handler class="mail-xml-body-handler">
        <xpath>/document/content</xpath>
        <content-type>text/plain</content-type>
      </body-handler>
    </mail-creator>
  </producer>
</standard-workflow>
{% endhighlight %}
</figure>
