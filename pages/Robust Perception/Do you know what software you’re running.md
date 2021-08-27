---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/do-you-know-what-software-youre-running
author: [[Brian Brazil]] 
---
# Do you know what software you’re running?

> When getting something working for the first time, it's easy to get caught up in Docker or Vargant. Before you run it in production with full access and user data, do you know what code you're running?

---
When getting something working for the first time, it's easy to get caught up in Docker or Vargant. Before you run it in production with full access and user data, do you know what code you're running?

Technologies like Docker, Vagrant, Maven, Pip, Apt and many others make it easy to try out new technologies and libraries. This is a boon to development velocity, but it comes with risks too. When there's a nasty bug you want to be able rebuild your system with a precisely targeted fix, without other things silently changing under the covers. In essence you need to know what software you're running, so that you can recreate it later on.

To start with, where is this software coming from and can you trust it? As a good example, the official Debian repositories are well run, and have been for a long time. Packages are built, cryptographically signed, distributed and the signatures checked before installation. If you try to install unsigned packages, you'll encounter friction. By contrast Pip only started [using HTTPS by default](https://github.com/pypa/pip/commit/9e40899e4892c9866a0b8dc636298c444565ddbb), [verifying HTTP certs](https://github.com/pypa/pip/pull/791) and [disallowing HTTP by default](https://github.com/pypa/pip/pull/1055) in 2013.

Let's assume though that there's no malicious interference with your downloads. Does that mean that everything is okay?

Not quite, another thing to consider is what version of the software you're using. If you pull down whatever the latest version is, how will you tell later on if that was version 1.1, 1.2 or even the head of a git repository? Does what you're using pull in any software itself, and if so what versions of that software is it pulling in?

[![](http://www.robustperception.io/wp-content/uploads/2015/11/Do-you-know-what-software-youre-running-2.png)](http://www.robustperception.io/wp-content/uploads/2015/11/Do-you-know-what-software-youre-running-2.png)

A Maven snapshot dependency could be buried deep below Vagrant

Each layer of software is something that you must tie down and have confidence in. It's a core feature that Vagrant and Dockerfile might run any code and thus download from the internet, but you may not have considered .deb preinstall scripts. For any software that's being used whether it is your own or third party you need to know which version you're using and that you're getting it from a reputable source. This applies recursively for any dependencies.

Being able to recreate your software from scratch is part of the first step to having [basic infrastructure](http://www.robustperception.io/do-you-have-basic-infrastructure/),
