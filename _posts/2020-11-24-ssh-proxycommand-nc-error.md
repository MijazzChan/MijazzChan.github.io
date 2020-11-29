---
title: SSH ProxyCommand NC ERROR
author: MijazzChan
date: 2020-11-24 21:51:37 +0800
categories: [Linux, Proxy]
tags: [linux,manjaro,notes,error,trace]
---



# SSH ProxyCommand NC ERROR

## Description

> Platform: Manjaro 20
>
> Configuration reference: [using-proxy-for-git-or-github.md](https://gist.github.com/coin8086/7228b177221f6db913933021ac33bb92)

```shell
Error: Couldn't resolve host "x.x.x.x:8888"
kex_exchange_identification: Connection closed by remote host
Connection closed by UNKNOWN port 65535
```

When I was configuring ssh proxy tunnel,`ProxyCommand nc ...` , then I try to `ssh -T git@github.com` to validate my proxy configuration. I got those errors described below.

It seems like `nc` didn't recognized the option `-X` , thus failed to use the proxy.

I tried to locate nc by using `which nc`, then I got `/usr/bin/nc`.

## Locate the problems

So I try to find to trace the version of `nc`, to see whether its version is too old or something with its dependencies causing it to be outdated.

> This kind of problem is not likely to happen on a rolling base system like `Manjaro`, since it always receives the latest softwares/dependencies update.

```shell
nc --version
netcat (The GNU Netcat) 0.7.1
Copyright (C) 2002 - 2003  Giovanni Giacobbi

This program comes with NO WARRANTY, to the extent permitted by law.
You may redistribute copies of this program under the terms of
the GNU General Public License.
For more information about these matters, see the file named COPYING.

Original idea and design by Avian Research <hobbit@avian.org>,
Written by Giovanni Giacobbi <giovanni@giacobbi.net>.

nc --help
GNU netcat 0.7.1, a rewrite of the famous networking tool.
Basic usages:
connect to somewhere:  nc [options] hostname port [port] ...
listen for inbound:    nc -l -p port [options] [hostname] [port] ...
tunnel to somewhere:   nc -L hostname:port -p port [options]
```

This is what i got when exec `nc --version` & `nc --help`

Then i try using `pamac search` to see whether the `/usr/bin/nc` in my path/system have some kind of dependency problems.

I got

```shell
pamac search netcat 
# Output
openbsd-netcat                                                                                                     1.217_2-1  community 
    TCP/IP swiss army knife. OpenBSD variant.
gnu-netcat                                                                                             [Installed] 0.7.1-8    extra 
    GNU rewrite of netcat, the network piping application
```

Turns out that there are two different versions of `nc`, one is under `openbsd`, the other one is under `gnu`.

## Solution

attempt installing the right version of `nc`

> It may prompt "To remove conflict", just type `y`, let `pacman` handle for you.

```shell
sudo pacman -S openbsd-netcat
```

```shell
nc -help
OpenBSD netcat (Debian patchlevel 1)
usage: nc [-46CDdFhklNnrStUuvZz] [-I length] [-i interval] [-M ttl]
          [-m minttl] [-O length] [-P proxy_username] [-p source_port]
          [-q seconds] [-s sourceaddr] [-T keyword] [-V rtable] [-W recvlimit]
          [-w timeout] [-X proxy_protocol] [-x proxy_address[:port]]
          [destination] [port]
```

It lists `-X` as `proxy_protocol`, just like [using-proxy-for-git-or-github.md](https://gist.github.com/coin8086/7228b177221f6db913933021ac33bb92) and [nc docs](https://linux.die.net/man/1/nc).

here is my `~/.ssh/config`

```
Host github.com
  User git
  IdentityFile "/path/to/key"
  ProxyCommand nc -X 5 -x 192.168.123.4:8888 %h %p
```

And it works

```shell
ssh -T git@github.com

Enter passphrase for key '/path/to/key*****': 
Hi ****! You've successfully authenticated, but GitHub does not provide shell access.
```

