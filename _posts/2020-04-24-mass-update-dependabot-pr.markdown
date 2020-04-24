---
title:             "Mass update of dependabot PRs"
description:       "What to do when a new log4j is released and everything has a new PR..."
published:         true
categories:        [github]
tags:              [github]
author:            [quotidian-ennui]
excerpt_separator: <!-- more -->
billboard: /billboards/icon-open-source.png
---

All our open source projects love [dependabot](https://dependabot.com) since this gives us a really easy way of keeping on top of version updates. What can be a little bit frustrating is when there's a whole bunch of pull requests because something that's relied on by many projects is updated. The usual culprit for us would be log4j / slf4j which causes a cascade of PR generation by dependabot.

<!-- more -->

Of course, you can script the hell out of this; it's a bit of shell scripting and making use of some opensource tooling that's already available. Since we know that we can use the github api to add comments (you can probably do this using the GraphQL variant if that's your bag, but since I'm not familiar with it, and Friday afternoon's aren't the best time to learn new things...); once we get the correct link, we can do a POST and create a new comment that tells dependabot to do its thing.

We typically don't have dependabot configured to auto-merge on approval; if we did it would be simpler: we would just use `repo approve log4j` to make everything happen.

* First of all install [https://github.com/googleapis/github-repo-automation](https://github.com/googleapis/github-repo-automation); and configure it

```
---
githubToken: YourPersonalAccessTokenWithSufficientRights
repos:
  - org: adaptris
    regex: interlok-.*
  - org: adaptris-labs
    regex: .*
```

* `repo list log4j` for instance will give you a bunch of data

```
Loaded 101 repositories from GitHub.
Total 101 unique repositories loaded.
Successfully processed: 45 pull request(s)
  https://github.com/adaptris-labs/interlok-soiltype-demo/pull/19    Bump log4j2Version from 2.13.1 to 2.13.2 in /builder
  https://github.com/adaptris-labs/interlok-hpcc-docker/pull/21      Bump log4j2Version from 2.13.1 to 2.13.2
  https://github.com/adaptris-labs/interlok-holodeck/pull/27         Bump log4j2Version from 2.11.2 to 2.13.2
....
```

* Now it's just a case of massaging the output so that you hit the correct URL and create a comment (subject to your API limits)....

```
AUTH_HEADER="Authorization: token YourPersonalAccessTokenWithSufficientRights"
TMPFILE=$(mktemp -t dependabot-XXXXX)
REGEX=$1
repo list $REGEX >$TMPFILE
PR_LIST=$(cat $TMPFILE | grep adaptris | awk '{ print $1}' | sed -e "s/pull/issues/g" | sed -e"s/$/\/comments/g" | sed -e "s/https:\/\/github.com\//https:\/\/api.github.com\/repos\//g")
for pr in $PR_LIST; do
  curl -si -H "$AUTH_HEADER" -X POST -d '{"body": "@dependabot merge"}' "$pr"
  # sleep 1
done
rm $TMPFILE
```