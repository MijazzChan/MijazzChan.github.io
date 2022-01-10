---
title: Import GPG Key
author: MijazzChan
date: 2020-11-24 17:15:02 +0800
categories: [Linux, other]
tags: [notes]
---

# Import GPG Key

After re-installing the system, I lost all my git setting since I did not back up the `gitconfig` file.

+ I have a 2 gpg keys, one for `Github` and one for my personal `Gitea Server`.

  ```bash
  ## To Find Keys available/active on system
  gpg --list-keys
  # Using the github key as the global option
  git config --global user.signingkey $MY_GITHUB_GPGKEY$
  # Project-wise setting 
  git config user.signingkey $MY_OWN_GPGKEY$
  ```


## How to re-import

+ locate the key first, in my case `privategpg.key` is the private gpg key

```shell
gpg --import ./privategpg.key
```

It will prompt for key password

```shell
# Terminal Output
gpg: directory '/home/mijazz/.gnupg' created
gpg: keybox '/home/mijazz/.gnupg/pubring.kbx' created
gpg: /home/mijazz/.gnupg/trustdb.gpg: trustdb created
gpg: key AA***********D: public key "MijazzChan <mijazz@vip.qq.com>" imported
gpg: key AA***********D: secret key imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

## Trust Ultimately

```shell
gpg --edit-key $GPG_KEY_EMAIL$

gpg> trust

5 -> Trust Ultimately
```



## Sign the commit

Once the key has been imported, simply `git commit -S ...` will make git sign the commit. You may be asked for private key password if your key was generated with it.

