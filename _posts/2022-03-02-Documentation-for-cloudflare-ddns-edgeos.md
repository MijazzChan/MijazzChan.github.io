---
title: "Documentation for cloudflare-ddns-edgeos"
date: "2022-03-02 16:32:09 +0800"
categories: [Linux, Management]
tags: [linux, shell, cloudflare-ddns-edgeos]
pin: true
---

## About this project

<p><a href="https://github.com/MijazzChan/cloudflare-ddns-edgeos/releases" class="img-link" style="display: inline-block !important;"><img data-src="https://img.shields.io/github/v/release/MijazzChan/cloudflare-ddns-edgeos?color=success&label=Latest&logo=github&style=for-the-badge" alt="GitHub release (latest by date)" src="https://img.shields.io/github/v/release/MijazzChan/cloudflare-ddns-edgeos?color=success&label=Latest&logo=github&style=for-the-badge"></a> <a href="https://github.com/MijazzChan/cloudflare-ddns-edgeos" title="" class="img-link" style="display: inline-block !important;"><img data-src="https://img.shields.io/github/stars/MijazzChan/cloudflare-ddns-edgeos?color=blue&label=STARS&logo=github&style=for-the-badge" alt="GitHub Repo stars" src="https://img.shields.io/github/stars/MijazzChan/cloudflare-ddns-edgeos?color=blue&label=STARS&logo=github&style=for-the-badge"></a> <a href="javascript:void(0)" title="" class="img-link" style="display: inline-block !important;"><img data-src="https://img.shields.io/badge/Made%20with-Bash-1f425f.svg?style=for-the-badge" alt="made-with-bash" src="https://img.shields.io/badge/Made%20with-Bash-1f425f.svg?style=for-the-badge"></a> <a href="http://www.wtfpl.net/about/" class="img-link" style="display: inline-block !important;"><img data-src="https://img.shields.io/badge/License-WTFPL-blue.svg?style=for-the-badge" alt="License: WTFPL" src="https://img.shields.io/badge/License-WTFPL-blue.svg?style=for-the-badge"></a> <a href="javascript:void(0)" title="" class="img-link" style="display: inline-block !important;"><img data-src="https://img.shields.io/badge/PRs-welcome-brightgreen?style=for-the-badge" alt="PR: Welcome" src="https://img.shields.io/badge/PRs-welcome-brightgreen?style=for-the-badge"></a></p>

Inspired by [rehiy/dnspod-shell](https://github.com/rehiy/dnspod-shell), I wrote this project. At the beginning, this is just a random script I wrote and configured it on my router with `crontab`, all because of my sudden switch of dns provider. 

There're plenty of ddns(`ddns-client`) projects on Github.

+ Some of them are written in `shell-script` and don't require extra language or framework support. (I don't want my poor router host a docker container everytime just for ddns update...)

+ Some of them are specifically optimized, ( or let's say ) , are only written for cloudflare ddns update, but in other language except `shell-script`.

Thus, [cloudflare-ddns-edgeos](https://github.com/MijazzChan/cloudflare-ddns-edgeos) aims at 

+ For routers running EdgeOS (Ubiquiti EdgeRouter), light-weighted design.

+ Written in `shell-script`, avoid complex language and framework dependencies. 

+ Specifically for Cloudflare DNS update, via Cloudflare v4 api.

And...EdgeOS didn't have official support for ddns on cloudflare, at least not an option in their web-ui. You need to configure it under cli, and it communicates with cloudflare through v1 api (currently v4 is actively supported & maintained by cloudflare)...

## Usage

### 0 - Prerequisite

Make sure you have following dependencies installed.

+ `curl`
+ `jq`
+ `ip`

### 1 - Download

> Suppose you have already ssh into your EdgeOS based router, in my case, Ubiquiti ER-X(v1.10.11).

#### 1.1 Using Github Raw

```bash
cd /config/scripts/ && pwd
# /config/scripts/

curl -LS -o /config/scripts/cloudflare-ddns-edgeos.tar.gz https://github.com/MijazzChan/cloudflare-ddns-edgeos/raw/releases/cloudflare-ddns-edgeos.tar.gz
```
{: file="terminal" }

#### 1.2 Trouble with GFW?

> [JsDelivr Github CDN](https://www.jsdelivr.com/github)

```bash
cd /config/scripts/ && pwd
# /config/scripts/

curl -LS -o /config/scripts/cloudflare-ddns-edgeos.tar.gz https://cdn.jsdelivr.net/gh/MijazzChan/cloudflare-ddns-edgeos@releases/cloudflare-ddns-edgeos.tar.gz
```
{: file="terminal" }


### 2 - Deploy

#### 2.1 - Extraction

```bash
tar -xvf /config/scripts/cloudflare-ddns-edgeos.tar.gz
```

#### 2.2 - Global Parameters Editing

**!!! This step is IMPORTANT !!!**

Alter variables in `/config/scripts/cloudflare-ddns-edgeos/cfddns.sh`

Meanings & where-can-be-found of global parameters is provided in `cfddns.sh` via code comments.

##### 2.2.1 - Cloudflare Access Token

Variable in `cfddns.sh` - `CF_ACCESS_TOKEN`

Visit [Cloudflare Api-tokens](https://dash.cloudflare.com/profile/api-tokens), Click `Create Token`.


![Cloudflare Access Token 1](/assets/img/blog/20220302/cf_token1.png)

Use Edit zone DNS template.

![Cloudflare Token Template](/assets/img/blog/20220302/cf_token2.png)

Fill in the form.

![Cloudflare Token Detail](/assets/img/blog/20220302/cf_token3.png)

**Continue to Summary**, and **Create Token**.

Copy following token string into `CF_ACCESS_TOKEN`, starting with `Bearer `.

![Cloudflare Token String](/assets/img/blog/20220302/cf_token4.png)

##### 2.2.2 - Zone Identifier

Variable in `cfddns.sh` - `ZONE_IDENTIFIER`.

![Zone Identifier](/assets/img/blog/20220302/zone_id.png)

##### 2.2.3 - Record Name

Variable in `cfddns.sh` - `RECORD_NAME`

Full domain name(With subdomain)

eg. subdomain.mijazz.icu, example.subdomain.yourdomain.com

#### 2.3 - Extra Steps

`+x` to files using `chmod +x /config/scripts/cloudflare-ddns-edgeos/cfddns.sh`

### 3 - Test 

Before you set it as a scheduled task, dryrun/execute it first. 
```bash
/config/scripts/cloudflare-ddns-edgeos/cfddns.sh
```
If it creates and updates your domain record **both successfully and correctly**, proceed to next step. 

**Otherwise**, feel free to [open a issue](https://github.com/MijazzChan/cloudflare-ddns-edgeos/issues/new)

### 4 - Using `task-scheduler` to `cron` it

```bash
configure

set system task-scheduler task cloudflareddns executable path /config/scripts/cloudflare-ddns-edgeos/cfddns.sh
set system task-scheduler task cloudflareddns interval 20m

commit
save
exit
```

## License

Copyright © 2022 MijazzChan <mijazz@qq.com>

This work is free. You can redistribute it and/or modify it under the
terms of the Do What The Fuck You Want To Public License, Version 2,
as published by Sam Hocevar. See http://www.wtfpl.net/ for more details.