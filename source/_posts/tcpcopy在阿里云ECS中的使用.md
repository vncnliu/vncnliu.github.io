---
title: tcpcopy在阿里云ECS中的使用
date: 2017-01-21 14:16:28
category:
- 生产力工具
tag:
- tcpcopy
---

tcpcopy是一个用于回放现网流量的工具，关于她的介绍和原理移步[tcpcopy](https://github.com/session-replay-tools/tcpcopy)。
官方文档也不多，别管理解多少看还是要看的。
tcpcopy的使用离不开tcpdump,离线模式下要用tcpdump抓取tcp包用于回放，在使用是也要用tcpdump进行抓包调试使tcpcopy能正常工作。
安装tcpcopy时需注意一点如果使用离线模式要在configure时加上--offline。
现在我已经用tcpdump -i eth0 tcp and dst port 38810 -w online.pcap从现网机器dump下来的tcp流量。
先在我的虚拟机上试一下，我是在一台机器上跑intercept和tcpcopy,用的虚拟机软件是VMware。网络使用的NAT模式
{% asset_img 1.png vmware network %}
先启动intercept
/usr/local/intercept/sbin/intercept -i eno16777736 -F 'tcp and src port 38810' -l intercept.log
查看日志没有错误
{% asset_img 2.png intercept.log %}
然后启动tcpcopy（参数意义可以通过tcpcopy -h 查看）
/usr/local/tcpcopy/sbin/tcpcopy -x 38810-192.168.95.130:38810 -s 192.168.95.130 -c 192.168.10.123 -i online.pcap -a 10 -n 5 -l tcpcopy.log



