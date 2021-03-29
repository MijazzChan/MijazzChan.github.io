---
title: Dockerize My Network
author: MijazzChan
date: 2020-03-29 16:51:49 +0800
categories: [NotForBlog, Personal_Notes]
tags: [notes, Dockerfile, docker, networking]
---

## Current network architecture

### Visual diagram

> Just a sketch.

![Network arch](/assets/img/blog/20210329/network.png)

So-called safe `LAN`, in which there will be  no unexpected public traffic inbound under normal circumstances. 

+ In local `LAN`, use `socks5` and `http` for `next_hop` in `v2ray`. (2 ports needed)
+ In openvpn nat `LAN`, use `http` with proxy automation script(pac) to proxy `gfw` network flow to `v2ray` .(1 port needed)
+ Manual Proxy certain traffic to docker container `unblocknetease`.(1 port needed)

Public network(In my case, i have a public ip addr and pre-configured ddns)

+ Use `vmess` inbound with `tls`. (1 port needed)
+ Inside `v2ray`, use `routing` to pass traffic to `unblocknetease`.(docker network port binding needed)

In this situation, I have to utilize/listen/bind at lease 4 to 5 ports to get a fully functional “gateway”.

## Modifications

Combine native-runtime `v2ray`  and docker container `unblocknetease` into one single docker container. Server as a “gateway” and Bind only 2 ports. 

+ Inbound 1: `http`
+ Inbound 2: `vmess`

All traffic redirected to `unblocknetease` will be handled by `v2ray`' s routing feature.

## Dockerfile

```dockerfile
ARG APLINE_VERSION=3
FROM alpine:${APLINE_VERSION}

LABEL maintainer="Mijazz_Chan"
LABEL version="1.0"

ARG V2RAY_VER="4.34.0"
ARG UNBLOCK_NETEASE_VER="0.25.3"

ARG SYSTEM_ARCH="linux-64"

# PRODUCTION ENV
ENV V2RAY_BINARY="https://github.com/v2fly/v2ray-core/releases/download/v"${V2RAY_VER}"/v2ray-"${SYSTEM_ARCH}".zip"
ENV UNBLOCK_NETEASE_SRC="https://github.com/nondanee/UnblockNeteaseMusic/archive/refs/tags/v"${UNBLOCK_NETEASE_VER}".zip"
ENV NODE_ENV=production
ENV NPM_REG="https://registry.npmjs.org"


# DEVELOPMENT ENV
#ENV V2RAY_BINARY="http://download.mijazz.xyz/v2ray-linux-64.zip"
#ENV UNBLOCK_NETEASE_SRC="http://download.mijazz.xyz/v0.25.3.zip"
#ENV NPM_REG="https://registry.npm.taobao.org"


# Due to known network issue, you may need to uncomment this line or change mirrors
#RUN sed -i 's/dl-cdn.alpinelinux.org/opentuna.cn/g' /etc/apk/repositories

RUN apk add --update nodejs npm supervisor && \
    wget -O /tmp/v2ray.zip $V2RAY_BINARY && \
    wget -O /tmp/v2ray.zip.dgst $V2RAY_BINARY.dgst && \
    expect_md5=`awk '/MD5/{print $2}' /tmp/v2ray.zip.dgst` && \
    current_md5=`md5sum /tmp/v2ray.zip | cut -d' ' -f1` && \
    if test "$expect_md5" != "$current_md5"; then echo "Corrupt File"; exit -2; fi && \
    echo -e "\n=====V2ray Fetched=====\n=====MD5 CHECKSUM MATCH=====\nExpecting: $expect_md5\nCurrent:   $current_md5\n" && \
    unzip -q -d /opt/v2ray /tmp/v2ray.zip && \
    wget -O /tmp/unblock_netease.zip $UNBLOCK_NETEASE_SRC && \
    unzip -q -d /opt/unblocknetease /tmp/unblock_netease.zip && \
    mv $(find /opt/unblocknetease -mindepth 1 -maxdepth 2) /opt/unblocknetease/ && \
    npm --registry $NPM_REG install --production /opt/unblocknetease && \
    rm -rf /tmp/* /var/cache/apk/*

COPY ./server.crt /opt/unblocknetease/server.crt
COPY ./server.key /opt/unblocknetease/server.key

VOLUME /conf

ENTRYPOINT ["supervisord","--nodaemon", "--configuration", "/conf/supervisord.conf"]

```

+ volume for `/conf` should contain a valid `v2ray` json config, `supervisord` config.
+ Upstream project `unblocknetease` does not provide a configurable certificate path, you must have to follow this [github issue](https://github.com/nondanee/UnblockNeteaseMusic/issues/48) to sign/generate your own certificates.
+ Certificates are being `COPY` into docker image during `docker build` procedure. 

### Files

```shell
 ~/Dev/otherproject/net-breach  tree .                [Mon Mar 29,06:36:40 PM]
.
├── Dockerfile
├── server.crt
└── server.key

0 directories, 3 files

 ~/Dev/otherproject/netbreach-conf  tree .            [Mon Mar 29,06:37:35 PM]
.
├── supervisord.conf
└── v2ray.conf

0 directories, 2 files
```

`supervisord.conf`

```
[supervisord]
nodaemon=true
logfile=/dev/stdout
logfile_maxbytes=0
loglevel=info

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock

[program:v2ray]
priority=2
command=/opt/v2ray/v2ray -c /conf/v2ray.conf
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
autorestart=unexpected
startsecs=10
exitcodes=23
startretries=3

[program:unblocknetease]
priority=1
directory=/opt/unblocknetease
command=node app.js -p 50000:50001 -o kuwo qq joox kugou -e https://music.163.com
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
startsecs=10
startretries=3
```

`v2ray.conf`

> project related config parts

```json
#OUTBOUND
{
            "tag": "netease",
            "protocol": "http",
            "settings":{
                "servers": [
                    {
                        "address": "127.0.0.1",
                        "port": 50000
                    }
                ]
            },
            "streamSettings": null,
            "mux": null
}
# ROUTING
{
                    "type": "field",
                    "outboundTag": "netease",
                    "domain": [
                        "music.163.com",
                        "music.126.net"
                    ]
},
```

## Cli

```shell
sudo docker build -t net-breach:0.1 $NET_BREACH_PATH

sudo docker run $ASSIGN_RUNTIME(rm/it/restart policy) -p $PORT_BINDING  -v /home/mijazz/Dev/otherproject/netbreach-conf:/conf net-breach:0.1
```

+ Port 50002 for `http` inbound.
+ Port 50003 for `vmess` inbound.

No extra port needed. Traffic routing is done by `v2ray` inside the container.

```shell
2021-03-29 18:55:38,043 INFO success: unblocknetease entered RUNNING state, process has stayed up for > than 10 seconds (startsecs)
2021-03-29 18:55:38,044 INFO success: v2ray entered RUNNING state, process has stayed up for > than 10 seconds (startsecs)
```

## Few extra notes

+ Container timezone inside `alpine` image is not set, or not correct. Normally this will not cause any issue, but since `vmess` protocol is being used, it is recommended that you should bind/configure a read-only host timezone file to container.
+ Proxy next hop can be reached via inbound traffic, `unblocknetease` is also fully functional during test.
+ The way I handle params is not recommended with docker build procedure. (Whatever...It works...)
+ Supervisord conf probably needs a little tweaking, combination with `http server` and `DOCKER HEALTHCHECK` is more elegant. (Although this primitive way works.......In case of network issues, just restart docker container via docker restart policy)

## Result

> Redirect ALL traffic by vmess protocol via public network inbound.
>
> iPhone - iOS 13, (jailbroken-not related), with certificate trust.

![Network arch](/assets/img/blog/20210329/ip.jpg)

![Network arch](/assets/img/blog/20210329/netease.jpg)

## References

+ [v2fly/v2ray-core](https://github.com/v2fly/v2ray-core)
+ [nondanee/UnblockNeteaseMusic](https://github.com/nondanee/UnblockNeteaseMusic)
+ [Docker Docs](https://docs.docker.com/reference/)

## Disclaimer

+ This post or project does NOT endorse, claim or imply that you could use this as a tool of bypassing government network censorship or anonymizing your network activities. 
+ If any unit or individual that the project may be suspected of violation of their rights, you should promptly notify and provide proof of identity and proof of ownership, I will remove the relevant content after receiving certification documents.

