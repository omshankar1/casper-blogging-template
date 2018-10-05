---
date: 2018-09-22T20:04:40.407Z
title: Some nifty yum commands
---

```console
yum provides /usr/sbin/semanage

[shankar@test ~]# sudo yum provides /usr/sbin/semanage
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.mirror.serversaustralia.com.au
 * extras: centos.mirror.serversaustralia.com.au
 * updates: centos.mirror.serversaustralia.com.au
policycoreutils-python-2.5-22.el7.x86_64 : SELinux policy core python utilities
Repo        : base
Matched from:
Filename    : /usr/sbin/semanage

policycoreutils-python-2.5-22.el7.x86_64 : SELinux policy core python utilities
Repo        : firstwavedev
Matched from:
Filename    : /usr/sbin/semanage
```

```console
[shankar@test ~]# sudo yum whatprovides /usr/sbin/semanage
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.mirror.serversaustralia.com.au
 * extras: centos.mirror.serversaustralia.com.au
 * updates: centos.mirror.serversaustralia.com.au
policycoreutils-python-2.5-22.el7.x86_64 : SELinux policy core python utilities
Repo        : base
Matched from:
Filename    : /usr/sbin/semanage



policycoreutils-python-2.5-22.el7.x86_64 : SELinux policy core python utilities
Repo        : firstwavedev
Matched from:
Filename    : /usr/sbin/semanage
```