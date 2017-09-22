---
layout: post
title:  "OSX VPN设置"
date:   2014-10-02 07:37:09 +0000
categories: 开发
tags: [科学上网]
---

用L2TP/IPsec VPN服务访问外网查资料，老有几个麻烦问题：

1. 在系统偏好设置中勾选“Send all traffic over VPN connection”之后，所有流量走VPN连接，一方面国内部分网站速度比较慢，另外访问有一些比较重要的网站时，心里也有些担心。
2. 我使用的VPN服务，每次连接上之后都会动态地配置DNS为Google的服务器，因为国内DNS污染的问题，这个设置虽然必要，但有时也需要修改。通过系统偏好手动修的设置，总会在断开VPN之后丢失，每次修改很麻烦。
3. 我用的联通网络，动态配置的DNS很垃圾，通过系统偏好设置的DNS，在连接过VPN之后也会丢失，重新配置成那个垃圾DNS。

在Linux和一些Unix系统下，通过设置/etc/resolv.conf能够对DNS配置，但OSX上这个文件是由系统动态生成的，OSX的系统配置文件可以通过scutil进行管理，配置文件位置为：

``` shell
/Library/Preferences/SystemConfiguration/preferences.plist
```

根据配置文件中的配置项来看，每个接口都有一个配置项，位于ROOT/NetworkServices下面。每个Network Service都有一个以UUID为key的配置项，每个Network Service都有一个UserDefinedName。在我的电脑上，可以看到这些用户定义名称包括：Wi-Fi，Bluetooth PAN, VPN (L2TP over IPsec), ... 参考这些配置的命名，这里将VPN的配置称为VPN Network Service配置，将电脑上默认的连接配置称为Primary Network Service配置。

# VPN Network Service国内外流量自动切换

OSX在建立ppp连接之后，会调用/etc/ppp/ip-up脚本，断开ppp连接之后会调用/etc/ppp/ip-down脚本，Linux也有同样地机制。因为L2TP/IPsec的隧道里面走ppp协议，因此这种类型的VPN连接建立之后，也会调用相应的脚本。我们可以利用这个脚本做一些处理，自动地让国内的流量走原来的连接，国外的流量走VPN。 我们只需要修改路由表，让IP包根据不同的目的地址，选择是否走VPN接口即可。这个可以利用开源工具chnroutes根据这个地址的数据生成接生成ip-{up,down}两个脚本。可以看到脚本里面会自动添加针对中国IP地址的路由项，让访问中国IP地址的IP包绕过VPN。 将这两个脚本放到/etc/ppp/文件夹下即可。 在VPN连接前后，可以通过执行下面的命令，查看路由表的变动。

``` shell
$ netstat -rn
```

# 自动修改VPN Network Service DNS

正常情况下，大家应该用不上这个。VPN的DNS虽然可以直接通过“系统偏好设置”进行修改，但每次重新连接VPN之后修改便丢失了，很麻烦。可以利用ip-up脚本在每次连接VPN后自动配置DNS。 基本办法可以参考这篇文章和Comments的内容，主要利用osx的scutil工具进行配置。我对文章中的脚本做了简单的修改，如下：

```shell
SERVICES=$(scutil --nc list | grep 'Connected' | awk '{print $3}')
 for SERVICE in $SERVICES
 do
 scutil <<< "
 d.init
 get State:/Network/Service/$SERVICE/DNS
 d.add ServerAddresses * 8.8.8.8 114.114.114.114
 set State:/Network/Service/$SERVICE/DNS
 quit
 "
 done
```

将其添加到ip-up脚本底部即可。

# 修改Primary Network Service DNS

Primary Network Service按说应该和VPN没什么关系，但我在“系统偏好设置”中设置的DNS，总在使用VPN后丢失，因此又可以在ip-up和ip-down里面添加下面的脚本，保证在使用VPN前后，Primary Network Service的DNS总是自己想要的值，对于获取Primary Service的ID，参考了[这篇文章](http://hints.macworld.com/article.php?story=20050621051643993)。

``` shell
#修改PIRMARY Network Service DNS
 PRIMARY_SERVICE=$(scutil <<< "
 open
 get State:/Network/Global/IPv4
 d.show
 quit
 " | grep 'PrimaryService' | awk '{print $3}')
 
 scutil <<< "
 d.init
 get State:/Network/Service/$PRIMARY_SERVICE/DNS
 d.add ServerAddresses * 114.114.114.114 114.114.115.115
 set State:/Network/Service/$PRIMARY_SERVICE/DNS
 quit
 "
```
需要注意的是，如果在ip-up脚本中使用了设置VPN DNS的脚本，需要将VPN DNS修改的脚本放在修改Primary Service DNS的脚本之后执行。  