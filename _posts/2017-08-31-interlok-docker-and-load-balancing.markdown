---
title:             "Interlok, Docker and Load balacing"
description:       "Running Interlok in a docker container with a load balancer."
date:              2017-08-31 08:00:00
categories:        [interlok, docker]
tags:              [interlok, docker]
author:            mcwarman
excerpt_separator: <!-- more -->
---

So you've just heard that Interlok is available on [docker hub][interlok-docker-hub] and now your thinking what next?! 

<!-- more -->

There are many options, but what this post will try and do is start to scratch the surface and explain the following:

* Building and running your first Interlok docker image
* Scaling using docker-compose
* Load balancing the scaled instance

## Prerequisites

The following prequisites are required:

* A knowledge of Interlok and its configuration files
* [Docker][install-docker] and [docker-compose][install-docker-compose] installed.

## Building and Running your first image

First things first the initial docker image, to do this we need some Interlok config and a `Dockerfile`. We'll be working with a the programming classic "Hello World", to do this in Interlok we'll be using the `jetty-message-consumer` consumer and the `payload-from-metadata` service.

<figure class="highlight">
  <figcaption class="g code-caption">config/adapter.xml</figcaption>
{% highlight xml %}
<adapter>
  <unique-id>hello-world</unique-id>
  <channel-list>
    <channel>
      <unique-id>channel</unique-id>
      <consume-connection class="jetty-http-connection">
        <port>8081</port>
      </consume-connection>
      <produce-connection class="null-connection" />
      <workflow-list>
        <standard-workflow>
          <unique-id>workflow</unique-id>
          <consumer class="jetty-message-consumer">
            <destination class="configured-consume-destination">
              <configured-thread-name>hello-world</configured-thread-name>
              <destination>/*</destination>
            </destination>
            <parameter-handler class="jetty-http-ignore-parameters"/>
            <header-handler class="jetty-http-headers-as-metadata"/>
          </consumer>
          <service-collection class="service-list">
            <services>
              <payload-from-metadata-service>
                <template><![CDATA[Hello World!]]></template>
              </payload-from-metadata-service>
            </services>
          </service-collection>
          <producer class="jetty-standard-response-producer">
            <unique-id>Send HTTP Response</unique-id>
            <status-provider class="http-metadata-status">
              <code-key>adphttpresponse</code-key>
              <default-status>OK_200</default-status>
            </status-provider>
            <response-header-provider class="jetty-metadata-response-headers">
              <filter class="remove-all-metadata-filter"/>
            </response-header-provider>
            <content-type-provider class="http-raw-content-type-provider">
              <content-type>text/plain</content-type>
            </content-type-provider>
            <send-payload>true</send-payload>
          </producer>
        </standard-workflow>
      </workflow-list>
    </channel>
  </channel-list>
</adapter>  
{% endhighlight %}
</figure>

<figure class="highlight">
  <figcaption class="g code-caption">Dockerfile</figcaption>
{% highlight docker %}
FROM adaptris/interlok:snapshot-alpine
ADD config /opt/interlok/config
EXPOSE 8081
{% endhighlight %}
</figure>

Now that we have that lets build and run it!

{% highlight console %}
$ docker build -t interlok/hello-world:snapshot .
$ docker run --rm -p 80:8081 --name interlok-hello-world interlok/hello-world:snapshot
{% endhighlight %}

Success! What the above commands will do is build an image using the `Dockerfile` in the current directory and tag it as `interlok/hello-world:snapshot` and then run it mapping the container port `8081` as `localhost:80`.

To test it and see whats happened:

{% highlight console %}
$ curl -s http://localhost/hello-world
Hello World
$ docker images
REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
interlok/hello-world                   snapshot            542feb3eb8d5        3 minutes ago       343MB
$ docker ps -a
CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS                     PORTS                                      NAMES
fab215bd069d        interlok/hello-world:snapshot   "/docker-entrypoin..."   58 seconds ago      Up 57 seconds              5555/tcp, 8080/tcp, 0.0.0.0:80->8081/tcp   angry_wescoff
{% endhighlight %}

To stop either run the following or Ctrl+C in the window:

{% highlight console %}
$ docker stop interlok-hello-world
{% endhighlight %}

## Scaling with docker-compose

Now we have an image that builds we can use docker-compose to scale it. 

<figure class="highlight">
  <figcaption class="g code-caption">docker-compose.yaml</figcaption>
{% highlight yaml %}
version: '3.2'
services:
  interlok:
    build:
      context: .
      dockerfile: Dockerfile
    image: interlok/hello-world:snapshot
    environment:
      JVM_ARGS: -Xmx256m
{% endhighlight %}
</figure>

To run it:

{% highlight console %}
$ docker-compose up --scale interlok=3
{% endhighlight %}

As we can see from the output we have three running instances of Interlok container. These instances are only reachable from within the containers so we can test they are working using the `docker-compose exec` command:

{% highlight console %}
$ docker-compose exec --index 1 interlok bash  -c "wget -qO-  http://localhost:8081/helloworld"
Hello World!
$ docker-compose exec --index 2 interlok bash  -c "wget -qO-  http://localhost:8081/helloworld"
Hello World!
$ docker-compose exec --index 3 interlok bash  -c "wget -qO-  http://localhost:8081/helloworld"
Hello World!
{% endhighlight %}

To stop either run the following from the same directory or Ctrl+C in the window:

{% highlight console %}
$ docker-compose down
{% endhighlight %}

## Load balancing

The instances can now be scales but they aren't reachable, what is needed is load balancer. Using the image [`dockercloud/haproxy`][dockercloud-haproxy] we can update our `docker-compose.yaml` and introduce one:

The load balancer is containerised version of haproxy, the magic happens when it dynamically generates the contents of the load balancers configuration with a combination of environment variables and the mounted `/var/run/docker.sock` file.

The configuration:

<figure class="highlight">
  <figcaption class="g code-caption">docker-compose.yaml</figcaption>
{% highlight yaml %}
version: '3.2'
services:
  interlok:
    build:
      context: .
      dockerfile: Dockerfile
    image: interlok/hello-world:snapshot
    environment:
      JVM_ARGS: -Xmx256m
      EXCLUDE_PORTS: 5555,8080
  lb:
    image: dockercloud/haproxy
    links:
      - interlok
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 80:80
{% endhighlight %}
</figure>

To run:

{% highlight console %}
$ docker-compose up scale interlok=3
{% endhighlight %}

To test:

{% highlight console %}
$ curl -s http://localhost/hello-world
Hello World
{% endhighlight %}

And to stop again either run the following from the same directory or Ctrl+C in the window:

{% highlight console %}
$ docker-compose down
{% endhighlight %}


## Taking it Further

Now if we wanted extend the config a little and return the hostname as a part of the "Hello World" message we could and would end up with responses looking something like this:

{% highlight console %}
$ curl -s http://localhost/hello-world
Hello World from 724ac45ded45!
$ curl -s http://localhost/hello-world
Hello World from 2171e7976497!
$ curl -s http://localhost/hello-world
Hello World from 6a6c8543f41f!
{% endhighlight %}

How this is done isn't covered here but the solution is available in a [github project][demo].

## Summary

Docker is powerful tool that can be used in development lifecycle or as a part of a production stack. Interlok has working use cases of both, personally I've moved most of my development work over to it, creating purpose built images for testing and product demonstrations.

The load balancer [`dockercloud/haproxy`][dockercloud-haproxy] is just another example of clever things docker can do for you.

[install-docker]: https://docs.docker.com/engine/installation/
[install-docker-compose]: https://docs.docker.com/compose/install/
[interlok-docker-hub]: https://hub.docker.com/r/adaptris/interlok/
[dockercloud-haproxy]: https://github.com/docker/dockercloud-haproxy
[demo]: https://github.com/mcwarman/interlok-load-balanced