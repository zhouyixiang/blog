---
layout: post
title:  "Linux Init System"
date:   2016-09-28 09:30:56 +0800
categories: 开发
tags: [linux, init, centos]
---

今天安装软件，整理了一些关于init system的资料。

# Linux Init System
* [Operating system service management (Wikipedia)](https://en.wikipedia.org/wiki/Operating_system_service_management)

  不同操作系统(Linux, Unix, macOS, Android, ...)采用的系统服务管理技术。
* [init (Wikipedia)](https://en.wikipedia.org/wiki/Init)
  * Research Unix-style/BSD-style

   系统初始化时执行脚本/etc/rc控制服务启动。支持site-specific /etc/rc.local以免修改/etc/rc。后来添加模块化支持，允许在/etc/rc.d文件夹中添加脚本。/etc/rc.d中的脚本如果有启动顺序要求，在脚本内用dependency tags显式指明，系统用rcorder脚本确定它们的排序。
  * SysV-style

   添加runlevels概念，系统在不同runlevels间切换时，执行启动或结束各个服务管理脚本。默认runlevels在/etc/inittab中指定。
* [BSD-style vs SysV-style](https://bbs.archlinux.org/viewtopic.php?id=47087)

  BSD:
  * Startup scripts are generally kept in /etc/rc.d/
  * A small number of files (/etc/rc.sysinit, /etc/rc.local, etc.) control the startup process

  Sys V:
  * Startup scripts are generally kept in /etc/init.d/
  * There are also a number of /etc/rcX.d/ directories -- one for every run-level (i.e. X represents 0 through 6 and S, so, 8 altogether)
  * The contents of each /etc/rcX.d/ directory is a collection of soft-links to scripts in /etc/init.d/
  * Each soft-link in a specific /etc/rcX.d/ directory is named so it will execute in the order of it's alphabetical relationship to the other soft-links

  可以看到BSD和SysV都可以将服务启动脚本放到/etc/rc.d文件夹下。SysV将脚本软连接到不同runlevel对应的/etc/rcX.d/文件中，启动顺序通过软连接文件名确定，而BSD的启动顺序在脚本中指定。

  > Example (ls -1 /etc/rc3.d/):

  > S05vbesave  <-- execs first

  > S10acpid

  > S10sysklogd

  > S10xserver-xorg-input-wacom

  > S11klogd

  > S12dbus

  > S12hal

  > ...

  > S98usplash

  > S99acpi-support

  > S99laptop-mode

  > S99rc.local

  > S99rmnologin  <-- execs last

  > This example was actually 39 lines long -- and just represents run-level #3.  In addition, /etc/rc.local executes after the target run-level is finished doing init.
  So, given Sys V's complex hierarchy -- spanning 8 run-levels by an average of N-number of daemons -- you can probably guess why so many people rave about Arch's BSD-style init schema ;-)

* [Upstart vs. Runit vs. Systemd vs. Circus vs. God](http://centos-vn.blogspot.jp/2014/06/daemon-showdown-upstart-vs-runit-vs.html)

 Where Upstart uses ptrace to watch the forking, systemd runs each daemon in a control group (requires Linux 2.6.24 or newer) from which it can not escape with any amount of forking. This allows easy resource limiting , both for forking and non-forking daemons, since control groups were made for this sort of thing.

* [RHEL 6 ditches System V init for Upstart](http://searchenterpriselinux.techtarget.com/tip/RHEL-6-ditches-System-V-init-for-Upstart-What-Linux-admins-need-to-know)
* [RHEL 7 Systemd ](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/chap-Managing_Services_with_systemd.html)
* [chkconfig](http://linuxcommand.org/man_pages/chkconfig8.html)
