---
title: "How to Delete Images From a Private Docker Registry"
date: 2018-12-01T15:31:53+03:00
lastmod: 2018-12-01T15:31:53+03:00
draft: false
keywords: ["docker", "docker registry", "docker private registry", "delete docker image", "ci pipeline", "jenkins pipeline"]
description: "delete image from docker private registry"
tags: ["docker", "devops", "docker registry"]
categories: ["devops"]
author: "Aziz Unsal"

# You can also close(false) or open(true) something for this content.

# P. S.comment can only be closed
comment: true
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false

# You can also define another contentCopyright.e.g.contentCopyright: "This is another copyright."

contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show

hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.

# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

Let's say you configured a private Docker registry for your in-house development workflow and you're using it in your daily development and/or CI/CD workflow intensively.

When I configured the registry on our server, I use the official Docker [image](https://hub.docker.com/_/registry/) and I chose a <a name="reg-easy-cfg">quick configuration</a> after pulling that image.

<!--more-->
You know what I mean;

``` bash
docker run -d -p 5000:5000 --restart always --name registry registry:2
```

So our private repository runs on a Docker container inside a VM.

After that our CI/CD workflow was easily integrated with the registry. And after successful builds, Jenkins, the CI Server, has been pushed tagged images to that repository.

It had been worked perfectly until I realized, thanks to Slack channel that triggered by Jenkins pipeline after failed builds, a project's build failed. It seemed that there was nothing wrong in the pipeline stages; compile, static code analysis, unit and integration tests were succeeded as they were planned in the Jenkinfile.

But the last stage, the deployment stage, which uploads Java artifacts to an Artifactory server and also pushes the tagged Docker image to the private registry, was failed!

![Failed Jenkins Pipeline Step](/blog/j-pipeline-dep-stg-err.png)

And here is the Jenkins logs for failed pipeline stage:

``` bash
[fr_web_api_master-A6Z7NH5CALTCIRZDOAJLWHD33UZWBUUNT7MPW2QLLTI3GEWN6CYQ] Running shell script
+ docker tag nevalabs/fr_web_api:3.3.0-alpha.74 192.168.1.121:5000/nevalabs/fr_web_api:3.3.0-alpha.74
[Pipeline] sh
[fr_web_api_master-A6Z7NH5CALTCIRZDOAJLWHD33UZWBUUNT7MPW2QLLTI3GEWN6CYQ] Running shell script
+ docker push 192.168.1.121:5000/nevalabs/fr_web_api:3.3.0-alpha.74
The push refers to repository [192.168.1.121:5000/nevalabs/fr_web_api]
0067dc9a03a3: Preparing
ce1c3e5ced31: Preparing
cf4176453649: Preparing
b42928ca3a48: Preparing
8b6f3539d178: Preparing
1232434e8e6d: Preparing
003d585c808a: Preparing
04d202a86d92: Preparing
f1dfa8049aa6: Preparing
79109c0f8a0b: Preparing
33db8ccd260b: Preparing
b8c891f0ffec: Preparing
f1dfa8049aa6: Waiting
79109c0f8a0b: Waiting
1232434e8e6d: Waiting
003d585c808a: Waiting
04d202a86d92: Waiting
b8c891f0ffec: Waiting
33db8ccd260b: Waiting
8b6f3539d178: Pushed
ce1c3e5ced31: Pushed
1232434e8e6d: Pushed
003d585c808a: Pushed
f1dfa8049aa6: Layer already exists
79109c0f8a0b: Layer already exists
33db8ccd260b: Layer already exists
b8c891f0ffec: Layer already exists
cf4176453649: Pushed
0067dc9a03a3: Pushed
04d202a86d92: Pushed
b42928ca3a48: Retrying in 5 seconds
b42928ca3a48: Retrying in 4 seconds
b42928ca3a48: Retrying in 3 seconds
b42928ca3a48: Retrying in 2 seconds
b42928ca3a48: Retrying in 1 second
b42928ca3a48: Retrying in 10 seconds
b42928ca3a48: Retrying in 9 seconds
b42928ca3a48: Retrying in 8 seconds
b42928ca3a48: Retrying in 7 seconds
b42928ca3a48: Retrying in 6 seconds
b42928ca3a48: Retrying in 5 seconds
b42928ca3a48: Retrying in 4 seconds
b42928ca3a48: Retrying in 3 seconds
b42928ca3a48: Retrying in 2 seconds
b42928ca3a48: Retrying in 1 second
b42928ca3a48: Retrying in 15 seconds
b42928ca3a48: Retrying in 14 seconds
b42928ca3a48: Retrying in 13 seconds
b42928ca3a48: Retrying in 12 seconds
b42928ca3a48: Retrying in 11 seconds
b42928ca3a48: Retrying in 10 seconds
b42928ca3a48: Retrying in 9 seconds
b42928ca3a48: Retrying in 8 seconds
b42928ca3a48: Retrying in 7 seconds
b42928ca3a48: Retrying in 6 seconds
b42928ca3a48: Retrying in 5 seconds
b42928ca3a48: Retrying in 4 seconds
b42928ca3a48: Retrying in 3 seconds
b42928ca3a48: Retrying in 2 seconds
b42928ca3a48: Retrying in 1 second
b42928ca3a48: Retrying in 20 seconds
b42928ca3a48: Retrying in 19 seconds
b42928ca3a48: Retrying in 18 seconds
b42928ca3a48: Retrying in 17 seconds
b42928ca3a48: Retrying in 16 seconds
b42928ca3a48: Retrying in 15 seconds
b42928ca3a48: Retrying in 14 seconds
b42928ca3a48: Retrying in 13 seconds
b42928ca3a48: Retrying in 12 seconds
b42928ca3a48: Retrying in 11 seconds
b42928ca3a48: Retrying in 10 seconds
b42928ca3a48: Retrying in 9 seconds
b42928ca3a48: Retrying in 8 seconds
b42928ca3a48: Retrying in 7 seconds
b42928ca3a48: Retrying in 6 seconds
b42928ca3a48: Retrying in 5 seconds
b42928ca3a48: Retrying in 4 seconds
b42928ca3a48: Retrying in 3 seconds
b42928ca3a48: Retrying in 2 seconds
b42928ca3a48: Retrying in 1 second
received unexpected HTTP status: 500 Internal Server Error
[Pipeline] }
[Pipeline] // withDockerRegistry
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] echo
Error occurred while pushing image to the registry: hudson.AbortException: script returned exit code 1
```

Yeah, lots of Retrying lines and finally aborted! So what was that *internal server error*?

After some digging on the server, I found out that there was no free space on the registry's host;

``` bash
@DOCKERREGISTRY:~$ df -h
Filesystem                       Size  Used Avail Use% Mounted on
udev                             3.9G     0  3.9G   0% /dev
tmpfs                            786M  8.8M  778M   2% /run
/dev/mapper/ubuntu1604--vg-root   57G   57G  0    100% /
tmpfs                            3.9G     0  3.9G   0% /dev/shm
tmpfs                            5.0M     0  5.0M   0% /run/lock
tmpfs                            3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sda1                        472M   57M  391M  13% /boot
tmpfs                            786M     0  786M   0% /run/user/1000
```

The first thing came to my mind as a solution, was to delete some of the older Docker images in the registry[^1]. Who cares the ancient images! So I was ready to run the [Docker Registry HTTP API V2] (https://docs.docker.com/registry/spec/api/#deleting-an-image).

Getting a list of the tags:

``` bash
curl http://192.168.1.121:5000/v2/nevalabs/fr_web_api/tags/list
```

Getting the manifests with the desired tags:

``` bash
curl -v http://192.168.1.121:5000/v2/nevalabs/fr_web_api/manifests/3.3.0-alpha.22 -H 'Accept: application/vnd.docker.distribution.manifest.v2+json'
```

And finally delete that image:

``` bash
curl -X DELETE http://192.168.1.121:5000/v2/nevalabs/fr_web_api/manifests/sha256:bf0573ad563716a3f8ff48bb118e5ebf3d7bbdb329dfe8665fa7982b81da31f3 -v 
```

And ...

``` json
{"errors":[{"code":"UNSUPPORTED","message":"The operation is unsupported."}]}

```

![Houston, we have a problem!](/blog/houston-we-have-5acf5f.jpg)

I mentioned about [quick configuration] (#reg-easy-cfg) for Docker private registry earlier in this post. Yeah, that's the problem because I didn't understand fully the registry environment variables and in my case the one below is crucial.

``` yaml
storage:
  delete:
    enabled: false
```

Now let's see how we can solve this problem step by step;

## Override the Registry Configuration

First, go inside the running registry container and change the existing, with default options, configuration file.

``` bash
vi /etc/docker/registry/config.yml
```

``` yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

We need to add

``` yaml
delete:
  enabled: true
```

under the existing `storage` section in the configuration.

## Restart the Docker Engine

Run `$ sudo systemctl restart docker.service` command on the host.

## Delete Some Images by Using HTTP API

Getting a list of the tags:

``` bash
curl http://192.168.1.121:5000/v2/nevalabs/fr_web_api/tags/list
```

Getting the manifests with the desired tags:

``` bash
curl -v http://192.168.1.121:5000/v2/nevalabs/fr_web_api/manifests/3.3.0-alpha.22 -H 'Accept: application/vnd.docker.distribution.manifest.v2+json'
```

Copy Docker-Content-Digest And finally delete that image:

``` bash
curl -X DELETE http://192.168.1.121:5000/v2/nevalabs/fr_web_api/manifests/sha256:bf0573ad563716a3f8ff48bb118e5ebf3d7bbdb329dfe8665fa7982b81da31f3 -v 
```

We should see output like below

``` bash

*   Trying 192.168.1.121...
* TCP_NODELAY set
* Connected to 192.168.1.121 (192.168.1.121) port 5000 (#0)
> DELETE /v2/nevalabs/fr_web_api/manifests/sha256:bf0573ad563716a3f8ff48bb118e5ebf3d7bbdb329dfe8665fa7982b81da31f3 HTTP/1.1
> Host: 192.168.1.121:5000
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 202 Accepted
< Docker-Distribution-Api-Version: registry/2.0
< X-Content-Type-Options: nosniff
< Date: Thu, 29 Nov 2018 17:22:41 GMT
< Content-Length: 0
< Content-Type: text/plain; charset=utf-8
<
* Connection #0 to host 192.168.1.121 left intact
```

## Check Disk Space

Run `df -h` command to see if we gain any disk space. Actually, you won't see free space gaining immediately.

## Run the Garbage Collector

``` bash
sudo docker exec -it nevalabs_docker_registry bin/registry garbage-collect /etc/docker/registry/config.yml
```

Done.

[^1]: Unfortunately, increasing the disk space was not an option due to some reasons.
