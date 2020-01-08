---
title:             "Testing Interlok interoperability with AWS"
description:       "Mocking AWS interactions with localstack"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            quotidian-ennui
billboard: /billboards/icon-iot.png
excerpt_separator: <!-- more -->
---

One of the habits that most of our team have is to try and run everything on their local machine as much as possible to test behaviour and logic before doing any kind of deployment to something that isn't directly within our control. It's habit more than anything, coupled with the fact that we often do a lot of work while we're not connected to the internet. Virtualisation, and container technologies like Docker are our best friends.

<!-- more -->

Recently we've been doing a lot of work with AWS, and one of the things that has been exceptionally useful is [localstack](https://github.com/localstack/localstack) which effectively gives you something that simulates an awful lot of the AWS environment in docker. We use this all the time while developing our solutions before doing any kind of test deployment as this proves some of the logic before going through a security audit.

With 3.9.1; Interlok supports producing to AWS Kinesis streams. The downstream activity off the back of the stream doesn't concern us, we are primarily engaged with making sure that we can provide data to the stream. In that regard, using localstack is a great substitute to having a prototype AWS environment with Kinesis configured. So here's a quick walkthrough of how to test Interlok with a localstack instance with Kinesis, we also use this technique with S3, SQS, and SNS by using `aws-custom-endpoint` within the connection configuration.

Once you're happy with your workflow logic and things, then you will need to change your connection configuration to remove your custom endpoints, or in this instance, to remove all the properties that could be defaulted (_KinesisEndpoint, KinesisPort, VerifyCertificate_, and perhaps _Region_) when you switch to a real AWS instance.

### Kinesis with localstack.

We can just start localstack using `docker-compose`. In this example, we start kinesis with `USE_SSL=1` since the AWS Kinesis Producer Library doesn't work with vanilla HTTP; this actually generates and uses a self-signed certificate which we will have to deal with through configuration. The second localstack image starts up S3/SQS/SNS in vanilla HTTP mode (since this is fine for the underlying AWS Java client); since we aren't actually using S3/SQS/SNS we don't need that service definition, but I have it so that everything I might need is up and running.

```
version: '3.2'
services:
  localstack_https:
    image: localstack/localstack
    environment:
      - SERVICES=kinesis
      - USE_SSL=1
    ports:
      #Management - http://localhost:8025
      # - "8025:8080"
      - "4568:4568"

  localstack_http:
    image: localstack/localstack
    environment:
      - SERVICES=s3,sqs,sns
    ports:
      #SNS - http://localhost:4575
      - "4575:4575"
      #SQS - http://localhost:4576
      - "4576:4576"
      #S3 - http://localhost:4572
      - "4572:4572"

```

Once the images are up and running you'll see the ports that kinesis is listening on (usually 4568); then we can create the kinesis streams using the AWS commandline interface; the examples here use `jq` to extract the bits we care about out of the JSON response.

```bash
ENDPOINT_URL="https://localhost:4568"
AWS_FLAGS="--endpoint-url=$ENDPOINT_URL --no-verify-ssl"
streamName="MyStream"
aws $AWS_FLAGS kinesis create-stream --stream-name "$streamName" --shard-count 1
shardId=$(aws $AWS_FLAGS kinesis list-shards --stream-name "$streamName" | jq -r ".Shards[0].ShardId")
echo $shardId
```

Once you know the shardID then you can get a shard iterator (note that this only lasts for about 10 minutes before it becomes invalid) and get records on the stream.

```bash
iteratorId=$(aws $AWS_FLAGS kinesis get-shard-iterator --stream-name $streamName --shard-id $shardId --shard-iterator-type LATEST | jq -r ".ShardIterator")
echo $iteratorId
aws $AWS_FLAGS kinesis get-records --shard-iterator "$iteratorId"
```

### Interlok to Kinesis

The simplest use-case is to have Interlok listening on HTTP and producing the payload into kinesis. We're actually making some assumptions about the behaviour here: since we are connecting to a localstack instance, then the AWS secret/access keys can have any value; and that we always get a `200 OK` back when the workflow finishes (this is the default behaviour if we never specify a `jetty-response-service` or similar).


```xml
<channel>
  <consume-connection class="jetty-embedded-connection"/>
  <produce-connection class="aws-kinesis-kpl-inline-connection">
    <config>
      <key-value-pair>
        <key>KinesisEndpoint</key>
        <value>localhost</value>
      </key-value-pair>
      <key-value-pair>
        <key>KinesisPort</key>
        <value>4568</value>
      </key-value-pair>
      <key-value-pair>
        <key>Region</key>
        <value>us-west-1</value>
      </key-value-pair>
      <key-value-pair>
        <key>VerifyCertificate</key>
        <value>false</value>
      </key-value-pair>
      <key-value-pair>
        <key>MetricsLevel</key>
        <value>none</value>
      </key-value-pair>
    </config>
    <credentials class="aws-static-credentials-builder">
      <authentication class="aws-keys-authentication">
        <access-key>aaa</access-key>
        <secret-key>aaa</secret-key>
      </authentication>
    </credentials>
  </produce-connection>
  <workflow-list>
    <standard-workflow>
      <consumer class="jetty-message-consumer">
        <destination class="configured-consume-destination">
          <destination>/api/kinesis</destination>
        </destination>
        <parameter-handler class="jetty-http-ignore-parameters"/>
        <header-handler class="jetty-http-ignore-headers"/>
      </consumer>
      <service-collection class="service-list"/>
      <producer class="aws-kinesis-stream-producer">
        <stream>myStream</stream>
        <partition-key>myPartition</partition-key>
      </producer>
      <unique-id>/api/kinesis</unique-id>
    </standard-workflow>
  </workflow-list>
  <unique-id>kinesis</unique-id>
</channel>
```

### Testing it all

To pull it all together; we can have a simple bash script that iterates on the kinesis stream and gets all the records, if you run it with `./kinesis-iterator.sh myStream create` then it will create the stream and then repeatedly poll the stream until killed via `ctrl-c` or similar.

```bash
#!/bin/bash

export PYTHONWARNINGS=ignore

ENDPOINT_URL="https://localhost:4568"
AWS_FLAGS="--endpoint-url=$ENDPOINT_URL --no-verify-ssl"

function newStream()
{
  local streamName=$1
  aws $AWS_FLAGS kinesis create-stream --stream-name "$streamName" --shard-count 1
}

function getShardId()
{
  local streamName=$1
  shardId=$(aws $AWS_FLAGS kinesis list-shards --stream-name "$streamName" | jq -r ".Shards[0].ShardId")
  echo $shardId
}

function getShardIterator()
{
  local streamName=$1
  iteratorId=$(aws $AWS_FLAGS kinesis get-shard-iterator --stream-name $streamName --shard-id $(getShardId $streamName) --shard-iterator-type LATEST | jq -r ".ShardIterator")
  echo $iteratorId
}

function getRecords()
{
  local streamName=$1
  local iteratorId=$2
  iteratorId=${iteratorId:=$(getShardIterator $streamName)}
  aws $AWS_FLAGS kinesis get-records --shard-iterator "$iteratorId"
}

function loopUntilKilled() {
  local streamName=$1
  iteratorId=$(getShardIterator "$streamName")
  while true; do
    getRecords myStream "$iteratorId"
    sleep 5
  done
}

STREAM_NAME=$1
ACTION=$2

if [ "$ACTION" == "create" ]
then
  newStream "$STREAM_NAME"
  ## Sleep a couple of seconds, since the stream creation takes time
  sleep 2
  aws $AWS_FLAGS kinesis list-streams
fi
loopUntilKilled $STREAM_NAME
```

Then its just a case of starting the interlok instance and posting some data to it using `curl` or similar. Eventually you will see the script output multiple base64 encoded payloads.
