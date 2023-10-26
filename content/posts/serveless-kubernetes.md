---
title: "Using Kubernetes as a Serverless Platform"
date: 2020-09-15T11:30:03+00:00
# weight: 1
aliases: ["/serveless-kubernetes"]
tags: ["serverless","kubernetes","kubeless"]
author: "Rafael Peroco"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This shows how to use Kubernetes as a Serverless Platform"
canonicalURL: "https://rafaelperoco.github.io/using-kubernetes-as-a-serverless-platform/"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://rafaelperoco.github.io/static/images/datacenter.jpg"
    # alt: "Empty Datacenter"
    caption: "Using Kubernetes as a Serverless Platform"
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

### Before You Start

This documentation assumes that the Kubernetes cluster has been created and that you have access to it.

The cluster used for implementation was a GKE 1.19

Change the URL to your test domain

#### Dependencies

- Kubernetes cluster
- nginx-ingress as ingress controller
- kubectl
- unzip
- apache-utils (Apache Benchmark package for load testing)

### Installing Kubeless

[![asciicast](https://asciinema.org/a/418534.svg)](https://asciinema.org/a/418534)

1. Export the environment variable by fetching the latest Kubeless version from the git repository.

    ```shell
    export RELEASE=$(curl -s https://api.github.com/repos/kubeless/kubeless/releases/latest | grep tag_name | cut -d '"' -f 4)
    ```

2. Execute the command to install Kubeless on the cluster

    ```shell
    kubectl create -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-non-rbac-$RELEASE.yaml
    ```

3. Check if the Kubeless controller is running correctly (Running)

    ```shell
    kubectl get pods -n kubeless
    ```

### Installing the Kubeless CLI

[![asciicast](https://asciinema.org/a/418535.svg)](https://asciinema.org/a/418535)

1. Export the environment variable to set the operating system.

    ```shell
    export OS=$(uname -s| tr '[:upper:]' '[:lower:]')
    ```

2. Execute the curl command to download and install the Kubeless CLI.

    ```shell
    mkdir -p ~/bin; curl -OL https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless_$OS-amd64.zip && unzip kubeless_$OS-amd64.zip mv bundles/kubeless_$OS-amd64/kubeless ~/bin; chmod +x ~/bin/kubeless
    ```

3. Check if the command was installed correctly

    ```shell
    kubeless -h
    ```

### Implementing a Sample Go Function

[![asciicast](https://asciinema.org/a/418693.svg)](https://asciinema.org/a/418693)

1. Clone the example repository

    ```shell
    git clone https://github.com/rafaelperoco/using-kubernetes-as-a-serverless-platform.git
    ```

2. Enter the directory and execute the command to create the Function

    ```shell
    cd using-kubernetes-as-a-serverless-platform/
    kubeless function deploy goserverless --cpu 100m --runtime go1.14 --handler helloget.Hello --from-file helloget.go --dependencies go.mod
    ```

3. Create the trigger that will be used to activate the Function created earlier

    ```shell
    kubeless trigger http create goserverless --function-name goserverless --hostname kubeless.rafaelperoco.dev --gateway nginx
    ```

4. Create the test Autoscale rule, allowing for simple load testing

    ```shell
    kubeless autoscale create goserverless --min 1 --max 1000 --metric cpu --value 1
    ```

5. Wait a few minutes for resource creation and test the endpoint

    ```shell
    curl --header "Host: kubeless.rafaelperoco.dev" --header "Content-Type:application/json" kubeless.rafaelperoco.dev --data '{"message": "Hello World from Kubeless"}'
    ```

### Simple Load Test

1. Run the load test with Apache Benchmark

    ```shell
    ab -n 50000 -c 1000 kubeless.rafaelperoco.dev/
    ```

2. Check if the nodes are scaling

    ```shell
    watch kubectl get nodes
    ```
