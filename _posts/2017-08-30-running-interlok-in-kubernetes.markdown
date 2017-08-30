---
title:  "Running Interlok in Kubernetes"
date:   2017-08-30 15:04:23
author: mcwarman
categories: [interlok, kubernetes, docker]
tags: [interlok, kubernetes, docker]
description: "Deploying Interlok in Kubernetes via Minikube"
excerpt_separator: <!-- more -->
keywords: "interlok, kubernetes, docker"

---

The goal of this post is guide you through the steps of deploying Interlok containers in Kubernetes. Minikube is used, Minikube is a tool that makes it easy to run Kubernetes locally. As we all know; Kubernetes is an open-source platform for automating deployment, scaling, and operations of application containers across clusters of hosts, providing container-centric infrastructure.

<!-- more -->

## Setting up your Minikube environment

 You'll already have docker installed so you can just follow the detailed instructions available to install from [Minikube & kubectl][install-minikube]. Once everything is installed you need to create a Minikube VM:

{% highlight console %}
$ minikube start --kubernetes-version="v1.5.2" --vm-driver="hyperv" --memory=1024 --hyperv-virtual-switch="NATSwitch" --v=7 --alsologtostderr
{% endhighlight %}


The above command assumes the following:

* You are using Hyper-V as your hypervisor (Windows 10 FTW)
* [You have a Virtual Switch set-up called NATSwitch.][vagrant-windows10-hyperv]

Now we need to set the `kubectl` context to use Minikube and then verify everything is in working order.

{% highlight console %}
$ kubectl config use-context minikube
$ kubectl cluster-info
$ minikube dashboard
{% endhighlight %}


## Creating your Interlok Config

For testing purposes we will create an Interlok instance using a jetty-message-consumer that returns the text "Hello World". First create a project directory called _interlok-kubernetes_ within that create another directory _config_ and create an adapter config file with the following contents:

{% highlight xml %}
<adapter>
  <unique-id>hello-world</unique-id>
  <channel-list>
    <channel>
      <unique-id>channel</unique-id>
      <consume-connection class="jetty-embedded-connection">
        <unique-id>jetty-embedded-connection</unique-id>
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
            <content-type-provider class="http-configured-content-type-provider">
              <mime-type>application/json</mime-type>
            </content-type-provider>
            <send-payload>true</send-payload>
          </producer>
        </standard-workflow>
      </workflow-list>
    </channel>
  </channel-list>
</adapter>
{% endhighlight %}


It's recommended to test the config is working as expected by starting your local Interlok instance and loading it.

## Creating your Docker Container image

In the created directory (`interlok-kubernetes`) create a new `Dockerfile`:

{% highlight text %}
FROM adaptris/interlok:snapshot-alpine

ADD config /opt/interlok/config
{% endhighlight %}

An image needs to created inside the minikube environment so let's point docker at minikube and build the image.

{% highlight console %}
$ eval $(minikube docker-env)
$ docker build -t hello-interlok:1.0.0 .
{% endhighlight %}

Of course you can customise how you build your docker image; since our hello-world application is very simple, we don't need any other additional libraries; for different ways to customise your docker container image check out the git project [docker-interlok-template][].

## Create a Deployment

Next we create a Kubernetes Pod. A Pod is group of one of more containers. We use the `kubectl run` commands

{% highlight console %}
$ kubectl run hello-interlok --image=hello-interlok:1.0.0 --port=8080
$ kubectl get deployments

NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-interlok   1         1         1            1           2m

$ kubectl get pods

NAME                              READY     STATUS    RESTARTS   AGE
hello-interlok-1091946548-qr64v   1/1       Running   0          37s

{% endhighlight %}


Once it's started, then we can view deployments using `kubectl get deployments` which gives you a list of all the deployments; view the pods using `kubectl get pods`. The output of both should reference _hello-interlok_ in one way or another. The important one here is the _pod name_; you'll be needing that later.


## Create a Service and expose it.

{% highlight console %}
$ kubectl expose deployment hello-interlok --type=LoadBalancer
$ kubectl get services
NAME             CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
hello-interlok   10.0.0.132   <pending>     8080:30082/TCP   6s
kubernetes       10.0.0.1     <none>        443/TCP          3h

$ minikube service hello-node
$ kubectl logs <pod_name>

INFO  [hello-world] [StandardWorkflow] message [d5c1dfcd-404f-4695-b79e-18e1d2fec03f] processed in [2] ms

{% endhighlight %}


References:

* [https://kubernetes.io/docs/tasks/tools/install-minikube/]()
* [https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/]()
* [https://blogs.msdn.microsoft.com/wasimbloch/2017/01/23/setting-up-kubernetes-on-windows10-laptop-with-minikube/]()


[install-minikube]: https://kubernetes.io/docs/tasks/tools/install-minikube/
[docker-interlok-template]: https://github.com/adaptris/docker-interlok-template
[vagrant-windows10-hyperv]: https://quotidian-ennui.github.io/blog/2016/08/17/vagrant-windows10-hyperv#create-a-nat-switch
