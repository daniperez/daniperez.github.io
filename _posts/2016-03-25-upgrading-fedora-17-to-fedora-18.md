---
layout: post
title: "Upgrading Fedora 17 to Fedora 18"
date: 2016-03-25 08:00:00 +0200
# categories: 
tags: [linux, fedora, start, update, X, upgrade, boot]
---

Even if you follow the steps in [here](http://fedoraproject.org/wiki/Upgrading_Fedora_using_yum#Fedora_17_-.3E_Fedora_18), you might encounter a couple of problems after upgrading Fedora from  17 to 18 version.

1st problem: Yum database corruption
==========================

The first problem arises as soon as you try to use yum in the just updated system. You'll get a message like:

``` shell
> sudo yum update
(...)
DB_VERSION_MISMATCH:  Database environment version mismatch
(...)
Error: rpmdb open failed
```
To fix that, you only need  to remove some files created by the former Fedora 17's yum:

    > sudo rm -i /var/lib/rpm/__db.00*

2nd problem: PolicyKit cannot start
========================

The second problem is that X won't start. I don't know exactly which error messages got me to the right Google search (see References) but the messages you can find in `/var/log/messages` might be:

    Sep 22 23:40:29 grumpy systemd[1]: Starting Authorization Manager...
    Sep 22 23:40:29 grumpy systemd[1]: polkit.service: main process exited, code=exited, status=1/FAILURE
    Sep 22 23:40:29 grumpy systemd[1]: Failed to start Authorization Manager.
    Sep 22 23:40:29 grumpy systemd[1]: Unit polkit.service entered failed state.
    
In `/var/log/Xorg.0.log` I could see something went wrong but no many clues, just an error about the number of displays out-numbering the number of the devices or the like (sorry, I cannot find a copy of the error message).

To fix that, reinstalling polkit and systemd would suffice:

    > sudo yum reinstall polkit systemd

I hope this post was useful for you!

References
========

* [Fix issues with polkit aka Authorization Manager after reboot](https://github.com/xsuchy/fedora-upgrade/pull/3)
