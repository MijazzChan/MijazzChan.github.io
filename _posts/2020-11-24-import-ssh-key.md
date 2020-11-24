---
title: Import SSH Key
author: MijazzChan
date: 2020-11-24 18:35:34 +0800
categories: [NotForBlog, Notes]
tags: [notes]
---

# Import SSH Key

## Key backup

`github.key` for ssh login/authentication on `github.com`

> for generating key, visit [Github Docs](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

## Re-import keys

```shell
# Start key agent in background
eval "$(ssh-agent -s)"  
## Output
	Agent pid 28022
```

```shell
ssh-add ./github.key
```

You may encounter problem like

```shell
ssh-add ./github.key    
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0777 for './github.key' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
```

+ Key file in my scenario, is copied from a NTFS file system(Windows Partition). Since the permission of key file in Linux/Unix System is pretty strict, so you will be unable to add it into your system when the permission is not `0600`.

```shell
chmod 600 ./github.key
ssh-add ./github.key
```

```shell
# Output
Enter passphrase for ./github.key: 
Identity added: ./github.key (mijazz@qq.com)
```

```shell
ssh -T git@github.com
```

```shell
The authenticity of host 'github.com (192.30.255.113)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,192.30.255.113' (RSA) to the list of known hosts.
Hi ******! You've successfully authenticated, but GitHub does not provide shell access.
```

## Configure SSH through proxy

Assume you have a local proxy tunnel 

+ Socks5 proxy : `socks5://x.x.x.x:x`
+ HTTPS proxy  : `https://x.x.x.x:x`

You can force you ssh client to send all traffic via given proxy.

By editing `~/.ssh/config`

> This is a host-wise setting. Only `github.com` will use this proxy

```
# File in ~/.ssh/config
Host github.com
  User git
  IdentityFile "/home/mijazz/.securedkeys/github.key"
  # Socks5
  ProxyCommand nc -X 5 -x 192.168.123.4:8888 %h %p
  # Https
  # ProxyCommand nc -X connect -x 192.168.123.4:8889 %h %p
```

In [nc docs](https://linux.die.net/man/1/nc) , you can find detailed usage of `nc`.

> nc -X 5     # For socks5 protocol
>
> nc -X connect  # For https protocol
>
> From nc docs:
>
> **-X** *proxy_version*
> Requests that **nc** should use the specified protocol when talking to the proxy server. Supported protocols are ''4'' (SOCKS v.4), ''5'' (SOCKS v.5) and ''connect'' (HTTPS proxy). If the protocol is not specified, SOCKS version 5 is used.

## Problem You May encounter

```shell
nc: invalid option -- 'X'
Try `nc --help' for more information.
kex_exchange_identification: Connection closed by remote host
Connection closed by UNKNOWN port 65535
```

See [SSH ProxyCommand NC Error](../ssh-proxycommand-nc-error/)