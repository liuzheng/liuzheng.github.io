---
title: "Mac docker-manchine 问题"
description: ""
categorys: 
tags: [docker]
---


最近卸载了原先的 boot2docker 用上了 docker 1.9.0 感觉很高大上有木有！！！

但是问题来了，运行 `docker-machine create --driver virtualbox default` 是 OK 的。

    docker-machine env default
    export DOCKER_TLS_VERIFY="1"
	export DOCKER_HOST="tcp://192.168.99.101:2376"
	export DOCKER_CERT_PATH="/Users/liuzheng/.docker/machine/machines/default"
	export DOCKER_MACHINE_NAME="default"
	# Run this command to configure your shell: 
	# eval "$(docker-machine env default)"

都是 ok 的

那么就运行 `eval "$(docker-machine env default)"`咯~

接下来就跟鬼一样了。。。尼玛为什么之后直接运行 `docker` 会报错！

    The server probably has client authentication (--tlsverify) enabled. Please check your TLS client certification settings: Get https://192.168.99.101:2376/v1.21/images/json: remote error: bad certificate

唉。。。这有多麻烦啊！！！

解决办法如下

    export DOCKER_OPTS="-H $DOCKER_HOST --tls --tlskey $DOCKER_CERT_PATH/server-key.pem    --tlscert $DOCKER_CERT_PATH/server.pem --tlsverify --tlscacert $DOCKER_CERT_PATH/ca.pem "
    alias docker="docker $DOCKER_OPTS " 

愉快的运行 docker 吧~

