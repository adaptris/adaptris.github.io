---
title:             "When to write custom code revisited"
description:       "Interlok is a bit like pick'n'mix; I prefer fizzy cola bottles, maybe you like flying saucers"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            quotidian-ennui
excerpt_separator: <!-- more -->
---

We pride ourselves on not having too much of an opinion about whether or not you should write code, or use configuration to achieve your integration. If you have, by instinct, a developer mindset then you'll naturally gravitate towards writing code because that's what you're most familiar with. This is something that we get quite a lot from new customers where their technical team are primarily developers. _I can do this in language X; but I can't in Interlok; why should we bother using it_. That's one way of looking at it; we prefer to look at it in terms of __what does Interlok give us that means we can spend more time on providing business value__. 

<!-- more -->

I was recently involved in a short timeframe delivery to achieve SQS to Zendesk (one way asynchronous) integration for a company. They were happily using their preferred language platform of choice to deliver a custom built integration, but deadlines were tight and they got me in to deliver the same functionality using Interlok as a backup. Since Zendesk has a public REST api and we already have an SQS consumer I was pretty confident that I could get the plumbing done in short order using only configuration until they specified some custom logic which I will use to illustrate the point that _you can compose your behaviour using both code and configuration at the same time_. 

The particular scenario where I mixed and matched to my heart's content was basically creating organisations within Zendesk based on the contents of an SQS message: 

* Receive the document as JSON from SQS
* _update or create_ the organisation in Zendesk (we're going to use the `organisations/create_or_update.json` REST endpoint).
* Make sure that a special user  with the `ID:12345` is added as an organisation member if it doesn't already exist as an organisation member.

If you look at the Zendesk API all of these things are possible using their REST endpoints; however there appears to be a bit of a gotcha: if you execute *create_or_update.json* it gives you the organisation back as the response, but this doesn't contain any memberships; you have to go and query them separately. It's perfectly possible to do this purely using Interlok configuration, but it would have been unwieldy and possibly hard to understand, so I decided quickly that the steps in the processing chain would be this:

* `sqs-consumer` to receive the message.
* `json-transformation` to transform the original JSON into the Zendesk format.
* `http-request-service` to POST the data into Zendesk.
* _custom class_ to query the organisation members, and if _12345_ doesn't exist as a member, then add it; making use of the [Zendesk java API provided by Cloudbees](https://github.com/cloudbees/zendesk-java-client).

 I thought I could actually do the whole _update-or-create and then add the user_ chain inside a custom class; but as it turns out the Java API doesn't currently have support for the `create_or_update.json` endpoint. Since the organisation will be the HTTP response from the create-or-update; we can simply use the custom class to interrogate that, and only add the user to the organisation if it doesn't exist.

 This means our configuration chain is how simply this (steps skipped for brevity)

 ```xml
<consumer class="amazon-sqs-consumer">
  <unique-id>receive-orgs</unique-id>
  <destination class="configured-consume-destination">
    <destination>${amazon.sqs.orgs.queue}</destination>
  </destination>
  <poller class="random-interval-poller"/>
  <reacquire-lock-between-messages>true</reacquire-lock-between-messages>
</consumer>
<json-transform-service>
  <unique-id>transform-to-zendesk-org</unique-id>
  <source-json class="string-payload-data-input-parameter"/>
  <mapping-spec class="file-data-input-parameter">
    <destination class="configured-destination">
      <destination>${adapter.transforms.dir.url}/organisation-jolt.json</destination>
    </destination>
  </mapping-spec>
  <target-json class="string-payload-data-output-parameter"/>
  <metadata-filter class="remove-all-metadata-filter"/>
</json-transform-service>
<apache-http-request-service>
  <unique-id>update-organisation</unique-id>
  <url>${zendesk.api.url}/organizations/create_or_update.json</url>
  <content-type>application/json</content-type>
  <method>POST</method>
  <response-header-handler class="apache-http-discard-response-headers"/>
  <request-header-provider class="apache-http-no-request-headers"/>
  <authenticator class="apache-http-dynamic-authorization-header">
    <username>${zendesk.user}</username>
    <password>${zendesk.api.key}</password>
  </authenticator>
</apache-http-request-service>
<zendesk-add-default-org-members>
  <unique-id>update-organisation-membership</unique-id>
  <connection class="shared-connection">
    <lookup-name>zendesk</lookup-name>
  </connection>
  <default-member-user-id>${zendesk.default.organisation.membership}</default-member-user-id>
</zendesk-add-default-org-members>
 ```

The custom class itself isn't actually that interesting; bearing in mind that the payload after `update-organisation` is the organisation that was either created or updated we can simply create the appropriate model out of it. I realise that the code isn't clever, and I could have used Optionals and Functions to be more lambda-ry; but this is custom code that is going to be handed over to people whose primary language of choise isn't java; so I've kept it simple.

```java
@Override
public void doService(AdaptrisMessage msg) throws ServiceException {
  try {
    Long memberID = Long.parseLong(msg.resolve(getDefaultMemberUserId()));
    zendeskClient.set(getConnection().retrieveConnection(ZendeskConnection.class).zendeskClient());
    Long orgId = toOrganization(msg).getId();
    boolean alreadyExists = false;
    for (OrganizationMembership membership : zendeskClient.get().getOrganizationMembershipsForOrg(orgId)) {
      if (memberID.equals(membership.getUserId())) {
        alreadyExists = true;
        break;
      }
    }
    if (!alreadyExists) {
      OrganizationMembership m = new OrganizationMembership();
      m.setOrganizationId(orgId);
      m.setUserId(memberID);
      zendeskClient.get().createOrganizationMembership(m);
    }
  } catch (Exception e) {
    throw ExceptionHelper.wrapServiceException(e);
  } finally {
    zendeskClient.remove();
  }
}

private Organization toOrganization(AdaptrisMessage msg) throws Exception {
  try (InputStream in = msg.getInputStream()) {
    ReadContext context = JsonPath.parse(in, jsonConfig);
    return context.read("$.organization", Organization.class);
  }
}
```

There were a few other workflows required for this project, but this one illustrates how you can mix and match custom code with standard configuration items.

### Conclusions and takeaway

The business only cares about a supportable piece of integration that works reliably and can be maintained by non-developers. You can opt to do this wholly in code; I'm betting that if you're that way inclined, you may not give a rat's ass about supportability; or you can compose the business value you need using standard Interlok configuration and judicious of custom code where it makes things simpler from a support perspective.