---
layout: post
title: "Fxxk AWS Elasticsearch service"
tagline: ""
description: ""
category: 
tags: [AWS]
---

# AWS Elasticsearch
I admmit AWS have awsome services in his provide. But the Elasticsearch service is not the one.

It cost too much money!

You can find the cost list at here: <https://aws.amazon.com/elasticsearch-service/pricing/>

Here have a limit `Volume size must be an integer between 10 and 100/512/1536.`

So, that cost too much when I have more than 1TB data.

And it have limit in the Elasticsearch's version, you know a new version about Elasticsearch will wait so long time until aws support.

No plugin can install.

...

Fine, I plan to build my own Elasticsearch cluster!

It is simple, 3 or more EC2 instance with my love type, you can reserve it, with a big EBS Volume( now the biggest size is 16384 GiB, but some system does not supported, you need to know ).

I'm using CoreOS, and using the docker to run Elasticsearch, actully I build my own docker image, you can use DockerHub.

Same guide with the docker run elasticsearch...

The awsome way is add a ELB before the elasticsearch cluster, port 80 to 9200, quite simple.

Now you will have a Elasticsearch just like AWS provided.

When you need to change something, you can completely independent.

More detail you can decide by you own, this page just provide an idea to save money.
