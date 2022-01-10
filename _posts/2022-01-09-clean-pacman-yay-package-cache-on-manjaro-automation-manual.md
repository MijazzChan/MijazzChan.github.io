---
title: "Clean Pacman/Yay Package Cache on Manjaro or Arch-based Distros (Automation/Manual)"
date: "2022-01-09 12:36:23 +0800"
categories: [Linux, Management]
tags: [linux, manjaro, pacman, yay]
---

## Intro

After several months of using Manjaro Linux(Arch Based Distro), I feel like getting attracted to its package manager `pacman` and its Rolling Release pattern.

Most of the time, Arch has vanilla packages (not as heavily modified as the Ubuntu/Debian based distros) that you can directly compile or install by just using `pacman`. It really save the day when you don't have to worry about some OS specific workaround. Tweaking/Messing with dotfile while configuring a freshly installed non-functional package is a nightmare to me.

Arch or AUR, often ships the newest/latest software with just-right-open-the-box experience.

However, with all these upsides, we will also face some downsides. 

## Reproduce the Scenario

I didn’t notice my disk usage until I have to meet a specific project requirements(machine learning) which needs installing `cuda` and `tensorflow` manually. Then I found out that my `/var/cache/pacman/pkg` ate up almost 20GB of my storage space, which results that my `/` barely have enough space to fit my new packages in. (~~Don’t buy Solid-State-Drive smaller than 512GB, and definitely don’t throw a Windows on it...~~)

Because I am using `pacman` and `yay` to manage my system packages and never bother to clean them manually. It grows up in  size. Browsing on [ArchWiki](https://wiki.archlinux.org/title/pacman), turns out that `pacman` will not automatically clean the package cache after installing/upgrading package in case users need to downgrade to a lower version of that. 

Although `pacman` do provide `cli` parameters to remove (un)installed package cache, 

> + `pacman -Sc` to clean cache of the uninstalled packages.
> + `pacman -Scc` to clean all cache despite their state of installing.

It’s still worth keeping 1 or 2 cached packages of each (un)installed software in case of emergency.

## Clean `pacman` cache

> Generally located in `/var/cache/pacman/pkg`. If altered, pass param `-c` or `--cachedir` to specify different cache directory.

With the help of `paccache`, we can easily achieve the goal/purpose mentioned above. 

> Note that `paccache` is now come from package called `pacman-contrib`, it originally ships with `pacman` till it became a tool component split into the `...-contrib` package alongside with `pacdiff`, `paclist`, `rankmirrors`, etc. See also [Community/pacman-contrib](https://archlinux.org/packages/community/x86_64/pacman-contrib/). 

**For documentation/man of `paccache`, click [here](https://man.archlinux.org/man/paccache.8)**.

**TLDR?** 

Check dependency first, find out whether `pacman-contrib` is installed or not.

```bash
pacman -Q pacman-contrib
```

To list out the cache that needs to be cleaned, follow steps below.

```bash
paccache -dvk2
# This line above list out the packages which are flagged
# to delete. (but keep 2 versions of each of them.)
# paccache --dryrun --verbose --keep 2

paccache -dvuk1
# This line above list out the already uninstalled packages 
# which are flagged to delete. (but keep 1 version of each of
# them.)
# paccache --dryrun --verbose --uninstalled --keep 1
```

To clean up the cache, follow steps below.

```bash
paccache -rvk2 && paccache -rvuk1
# paccache --remove --verbose --keep 2 
# paccache --remove --verbose --uninstalled --keep 1
# which does the same trick.
```

## Clean `yay` cache

>  Generally located in `/home/$(whoami)/.cache/yay`. If altered, pass param `-c` or `--cachedir` to specify different cache directory.

Using the same `paccache` approach mentioned above can also be useful when it comes to `yay` cache. Point the cache directory to `/home/$(whoami)/.cache/yay/*/` by passing `-c /home/mijazz/.cache/yay/*/`(in my case).

Also, to list’em out

```bash
paccache -c /home/$(whoami)/.cache/yay/*/ -dvk2
# paccache -cachedir /home/$(whoami)/.cache/yay/*/ --dryrun --verbose --keep 2

paccache -c /home/$(whoami)/.cache/yay/*/ -dvuk1
# paccache -c /home/$(whoami)/.cache/yay/*/  --dryrun --verbose --uninstalled --keep 1
```

to clean’em

```bash
paccache -c /home/$(whoami)/.cache/yay/*/ -rvk2
# paccache -cachedir /home/$(whoami)/.cache/yay/*/ --remove --verbose --keep 2

paccache -c /home/$(whoami)/.cache/yay/*/ -rvuk1
# paccache -c /home/$(whoami)/.cache/yay/*/  --remove --verbose --uninstalled --keep 1
```

### `yay`'s  approach to clean

> Not recommended. (Interactive prompt in shell.)

```bash
yay -Scc

Cache directory: /var/cache/pacman/pkg/
:: Do you want to remove ALL files from cache? [y/N] N

Database directory: /var/lib/pacman/
:: Do you want to remove unused repositories? [Y/n] N

Build directory: /home/mijazz/.cache/yay
==> Do you want to remove ALL AUR packages from cache? [Y/n] y
removing AUR packages from cache...

```

## Achieve Automation via Hooks

Automatically clean cache after each package transaction can be achieved given you are familiar with `alpm-hooks`. 

It provides the ability to run hooks before or after package transaction based on the packages and/or files being modified. 

You can custom the trigger of the hook whether it will run `[ before | after ]` the `[ specific | * ]`package `[ Install | Upgrade | Remove ]` transaction.

In this specific scenario this post focusing on, a **cleaning-hook** triggered **after** package wildcard *****  being **upgraded** or **uninstalled** will be perfectly suitable.

> Hooks are written in alpm hook file format, [ArchWiki - alpm-hooks](https://man.archlinux.org/man/alpm-hooks.5)
>
> Default hook files location is `/usr/share/libalpm/hooks`,  and `/etc/pacman.d/hooks`, additionally it can be customized via `/etc/pacman.conf` - `HookDir`.
>
> ```
> #
> # /etc/pacman.conf
> #
> # See the pacman.conf(5) manpage for option and repository directives
> 
> #
> # GENERAL OPTIONS
> #
> [options]
> # The following paths are commented out with their default values listed.
> # If you wish to use different paths, uncomment and update the paths.
> #RootDir     = /
> #DBPath      = /var/lib/pacman/
> CacheDir = /var/cache/pacman/pkg/
> #LogFile     = /var/log/pacman.log
> #GPGDir      = /etc/pacman.d/gnupg/
> #HookDir     = /etc/pacman.d/hooks/
> ```
>
> More details can be found at [ArchWiki - Pacman #Hooks](https://wiki.archlinux.org/title/Pacman#Hooks)

Start with cleaning script. Name it `/home/$(whoami)/.local/bin/cleaning-automation`~~(whatever)~~, just don’t forget to `chmod +x`.

```bash
#!/usr/bin/env bash

# Clean cache upgrade/uninstall at PostTransaction
# /home/mijazz/.local/bin/cleaning-automation
# Hook starts here

# Suppose you run yay with your user account id 1000
yay_running_on_behalf=$(id -nu 1000)
yay_cache_dir="/home/$yay_running_on_behalf/.cache/yay/"

pkg_cache_dir="$(find $yay_cache_dir -mindepth 1 -maxdepth 1 -type d | xargs -r printf "-c %s ")"

echo "Removing cached packages, keeping latest 2 versions of each one."
/usr/bin/paccache -rk2
/usr/bin/paccache -rk2 $pkg_cache_dir

echo "Removing cached uninstalled packages, keeping latest 1 versions of each one."
/usr/bin/paccache -ruk1
/usr/bin/paccache -ruk1 $pkg_cache_dir
```
{: file="cleaning-automation" }

then the hook. Place it under `/etc/pacman.d/hooks`(if this specific directory doesn’t exist, just `mkdir` it). **Do remember that hook’s filename must look like `*.hook`**.

> File SYNOPSIS:
>
> ```
> [Trigger] (Required, Repeatable)
> Operation = Install|Upgrade|Remove (Required, Repeatable)
> Type = Path|Package (Required)
> Target = <Path|PkgName> (Required, Repeatable)
> [Action] (Required)
> Description = ... (Optional)
> When = PreTransaction|PostTransaction (Required)
> Exec = <Command> (Required)
> Depends = <PkgName> (Optional)
> AbortOnFail (Optional, PreTransaction only)
> NeedsTargets (Optional)
> ```
>
> 

```
# Pacman Cache Cleaning Hook

# /etc/pacman.d/hooks/cleaning-automation.hook

[Trigger]
Operation = Upgrade
Operation = Remove
Type = Package
Target = *

[Action]
Description = Pacman Cache Cleaning Hook
When = PostTransaction
Exec = /home/mijazz/.local/bin/cleaning-automation
Depends = pacman-contrib
```
{: file="cleaning-automation.hook"}

##  Conclusion

Using `PostTransaction` hook to achieve this automation is perfectly suitable for those folks who don’t have big `/` disk space but daily drive Arch based Linux. However, keeping extra versions of cache packages also benefits Linux user to some extend, in case you need to roll back some packages to previous version in order to prevent crashing or mis-behavior of the latest one.
