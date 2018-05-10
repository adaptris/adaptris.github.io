---
layout: post
title: "Interlok as a Jira Webhook"
author: [quotidian-ennui, mcwarman]
# date: 2018-01-15 11:00
comments: false
tags: [interlok, interlok-hammer]
categories: [interlok, interlok-hammer]
published: true
description: "Using Interlok as a Jira Webhook endpoint"
keywords: "interlok"
excerpt_separator: <!-- more -->
---

We've been using Jira for a while now, and they have pretty good integrations with other cloud services. If you're an MS Teams user, then you can have your own webhook to publish into a channel. Rather than using Jira's own connector for Teams; we thought it might be a nice idea to use Interlok to hook up Jira with Teams; yes it's ![Interlok Hammer](https://img.shields.io/badge/certified-interlok%20hammer-red.svg) time again
.
<!-- more -->

If we break down the problem into its constituent parts, then it's a standard integration problem. Receive input file in one format; map/enrich the data; publish it it somewhere else. All we need is a public facing Interlok instance and then we're good to go. We first started off with exactly this; configuring a webhook in Jira for capturing events for a given project. All the Interlok instance did was to save the data to disk so that we would have plenty of sample data to work with. While the [jira documentation](https://developer.atlassian.com/server/jira/platform/webhooks/) is quite comprehensive it doesn't have a lot of example data.

Once we captured enough example data then we're ready to do the work on mapping the data. Nothing beats example data from a real system.

## Mapping the data

We decided to use [jolt](https://github.com/bazaarvoice/jolt) via our own [optional json component](http://interlok.adaptris.net/interlok-docs/cookbook-json-transform.html) to do the legwork; we decided to map the incoming JSON into an intermediate format, and from there into the required teams format. The reasoning is two-fold; we are discarding a lot of the data as it's not really required for our notifications, and also because of adaptive cards. Adaptive cards are currently in preview, and we wanted to reduce the work if we were to support it.

So mapping an issue_created event means we just want to capture some key bits of information.

* The ticket number
* The description of the issue.
* The project it came from
* The person that created the issue

We can default the other pertinent bits of information.

```json
{
  "operation": "shift",
  "spec": {
    "user": {
      "displayName": "data.facts.[4].value"
    },
    "issue": {
      "key": "data.facts.[1].value",
      "@key": "data.jiraIssue",
      "fields": {
        "project": {
          "name": "data.facts.[0].value"
        },
        "summary": "data.text",
        "assignee": {
          "displayName": "data.facts.[3].value"
        },
        "status": {
          "name": "data.facts.[2].value"
        }
      }
    }
  }
},
{
  "operation": "default",
  "spec": {
    "data": {
      "facts[]": {
        "0": {
          "name": "Project:"
        },
        "1": {
          "name": "Issue:"
        },
        "2": {
          "name": "Status:",
          "value": "Unknown"
        },
        "3": {
          "name": "Assignee:",
          "value": "None"
        },
        "4": {
          "name": "User:",
          "value": "Nobody"
        }
      }
    }
  }
}]
```

After we have our intermediate format; we can now map it onto one of the MessageCard formats for MS Teams. Note that a lot of this is hard-coded as it's really a _gedankenexperiment_ to see how easy it would be. If you wanted more/less information then the complexity of your jolt transforms would change.

```json
[{
  "operation": "modify-overwrite-beta",
  "spec": {
    "data": {
      "jiraSummary": "=concat('Jira #', @(1,jiraIssue), ' Updated')",
      "jiraLink": "=concat('https://adaptris.atlassian.net/browse/', @(1,jiraIssue))"
    }
  }
},
{
  "operation": "shift",
  "spec": {
    "data": {
      "jiraSummary": "summary",
      "@jiraSummary": "title",
      "facts": "sections.[0].&",
      "text": "sections.[0].&",
      "jiraLink": "potentialAction.[].targets.[].uri"
    }
  }
},
{
  "operation": "default",
  "spec": {
    "@type": "MessageCard",
    "@context": "http://schema.org/extensions",
    "sections[]": {
      "0": {
        "activityTitle": "Jira Issue Updated",
        "activitySubtitle": "I have an Interlok hammer; this is a nail",
        "activityImage": "https://development.adaptris.net/images/adaptris-logo.png"
      }
    },
    "potentialAction[]": {
      "0": {
        "@type": "OpenUri",
        "name": "View Issue",
        "targets[]": {
          "0": {
            "os": "default"
          }
        }
      }
    }
  }
}]

```

### Naming the mappings

If we name the mappings so that they match the webhook events declared in jira then everything is pretty dynamic (`jira:issue_created` would have to mapped to `jira_issue_created` for filesystem safety) when you configure your transform services.

```xml
<json-transform-service>
  <mapping-spec class="file-data-input-parameter">
    <destination class="configured-destination">
      <destination>file://localhost/path/to/transforms/%message{webhookEvent}.json</destination>
    </destination>
  </mapping-spec>
</json-transform-service>
<json-transform-service>
  <mapping-spec class="file-data-input-parameter">
    <destination class="configured-destination">
      <destination>file://localhost/path/to/transforms/intermediate-to-ms.json</destination>
    </destination>
  </mapping-spec>
</json-transform-service>
```

## Making it project neutral

If we didn't want to have to setup a new Jira webhook for every project (we generally restrict how/when webhooks are triggered via JQL) then we need a database that maps the project to a Teams webhook URL. The easiest way to do this is with a CSV file and then use [csvjdbc](http://csvjdbc.sourceforge.net/) to treat it as a database. That's pretty simple thing to do then. If we have a file called `teams.csv` in the directory _csvjdbc_ and it contains simply

```
PROJECT_NAME,TEAMS_URL
Interlok,https://some.office.outlook.com/
```

Then we can simply do a select based on the project name and have the correct Teams URL stored against `TEAMS_URL`.

```xml
<jdbc-data-query-service>
  <unique-id>Extract Teams URL</unique-id>
  <connection class="jdbc-connection">
    <driver-imp>org.relique.jdbc.csv.CsvDriver</driver-imp>
    <connect-url>jdbc:relique:csv:./path/to/directory/containing/csv/files</connect-url>
    <always-validate-connection>false</always-validate-connection>
  </connection>
  <statement-creator class="jdbc-configured-sql-statement">
    <statement>SELECT * FROM teams where PROJECT_NAME = '%message{jiraProject}'</statement>
  </statement-creator>
  <result-set-translator class="jdbc-first-row-metadata-translator">
    <metadata-key-prefix></metadata-key-prefix>
    <separator></separator>
  </result-set-translator>
</jdbc-data-query-service>
```

## Wrapping Up

After that, all you need to do is to publish the existing message to the specified URL; and you'll end up with a action card.

```xml
<apache-http-request-service>
  <url>%message{TEAMS_URL}</url>
  <method>POST</method>
  <content-type>application/json</content-type>
</apache-http-request-service>
```


All of this is published into [github](https://github.com/quotidian-ennui/interlok-jira-msteams) so feel free to fork and bastardise it to your hearts content.


