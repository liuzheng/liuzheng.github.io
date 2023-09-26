---
layout: post
title: "istio sidecar 出向劫持"
tagline: ""
description: ""
category: istio
tags: [kubernetes, istio]
---

istio 方案在方案设计上和网络讨论热点都是入向策略，在出向方面聊的不多，且官方给的出向方案让我难以接受。

在此记录一下个人使用的出向方案。

## Nginx 代理
首先编译出一份含有 ngx_http_proxy_connect_module 的 nginx，过程不赘述。

nginx 的配置正向代理(具体优化以及实际业务配置请自行调整)：

```
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log error;
pid        /var/run/nginx.pid;
events {
	worker_connections  1024;
}
stream {
# tcp forward proxy by SNI
	server {
		resolver 8.8.8.8 ipv6=off;
		listen       8443;
		proxy_pass   $ssl_preread_server_name:443;
		ssl_preread  on;
	}
}

http {
	server {
		listen                         8080;

# dns resolver used by forward proxying
		resolver                       8.8.8.8;

# forward proxy for CONNECT request
		proxy_connect;
		proxy_connect_allow            443 563;
		proxy_connect_connect_timeout  10s;
		proxy_connect_read_timeout     10s;
		proxy_connect_send_timeout     10s;

# forward proxy for non-CONNECT request
		location / {
			proxy_ssl_session_reuse off;
			proxy_set_header Host $host;
			proxy_pass $scheme://$host$request_uri;
		}
	}
}
```
假定该机器的 IP 是 10.0.0.1

kubernetes 里添加 nginx service
```
apiVersion: v1
kind: Service
metadata:
  name: http-proxy
metadata:
  name: http-proxy
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: http-proxy
subsets:
  - addresses:
      - ip: 10.0.0.1
    ports:
      - port: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: https-proxy
metadata:
  name: https-proxy
spec:
  type: ClusterIP
  ports:
    - port: 8443
      targetPort: 8443
      protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: https-proxy
subsets:
  - addresses:
      - ip: 10.0.0.1
    ports:
      - port: 8443
```

这里有个坑，后面会讲。

## istio 搭建
这个也不赘述，并不需要 istio-egressgateway，此处由上面搭建的 nginx负责了

## istio VirtualService 
官方的方案太复杂，且对于tls不够友好，且对泛域名的配置不友好，此处的泛域名其实是对域名的一个预告，不是通配符的概念，
以 qq.com 为例，我们配置 `*.qq.com`，其含义代表所有二级qq.com的域名，并不包含三级域名，且如需配置三级泛域名，需要把其二级域名表示出来，
并不支持诸如 `*.*.qq.com`、`**.qq.com`之类的表示方法

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: qq-com-through-nginx-proxy-gateway
spec:
  hosts:
    - "*.qq.com"
  https:
  - route:
    - destination:
        host: http-proxy.default.svc.cluster.local
        port:
          number: 8080
    match:
      - port: 80
  tls:
  - route:
    - destination:
        host: https-proxy.default.svc.cluster.local
        port:
          number: 8443
    match:
    - port: 443
      sniHosts:
      - "*.qq.com"
---
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: qq
spec:
  hosts:
  - "*.qq.com"
  ports:
  - number: 443
    name: tls
    protocol: TLS
```
一切配置妥当，curl起来

## 坑
上面已经提及泛域名的事情了，下面聊聊非标tls端口的事情。

因为某些原因，确实有很多端口开启了https，针对这些开启了 tls 的端口，在做处理的时候就有点头疼了。
主要原因还是在SNI上，上面我也没细讲SNI，istio在做代理转发的时候是4层转发，由于https的包都是加密的，
也就是说转发的目的地只能在SNI上拿到，这里有个坑，SNI里只有地址，没有端口。
这里再注意一下nginx上面提及的坑，可以看到nginx的配置里这一行 `proxy_pass   $ssl_preread_server_name:443`，
对于nginx来说，所有经过4层的代理都被扔到了443端口上去了，有时候这并不是我们期望的。所以，这里需要再起一份nginx（建议），
利用nginx $server_port 这个变量来传。
```
stream {
# tcp forward proxy by SNI
	server {
		resolver 8.8.8.8 ipv6=off;
		listen       9443 8443...;
		proxy_pass   $ssl_preread_server_name:$server_port;
		ssl_preread  on;
	}
}
```

再在 kubernetes 里建立对应的 service ，对于特殊情况就转到这个nginx上去，
当然你可以偷懒，把443也写进去，这样就不用动了。

个人建议另起，毕竟上一份nginx正常情况一下已经投产稳定使用了，非标端口毕竟极少数且存在频繁修改的可能。

写在最后：这个东西想象空间还是蛮大的。
