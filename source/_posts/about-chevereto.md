---
title: 关于图床
date: 2021-02-21 20:35:46
---

试着建了个图床w

<!-- more -->

试了下用[docker-chevereto](https://github.com/linuxserver/docker-chevereto)和rclone来建了一个[图床](https://img.anillc.cn)  

用rclone的原因是为了使用巨硬的e5开发者计划  

遇到的问题是貌似rclone mount用到了fuse用网络跟rclone通信？具体原理暂时还没时间研究，反正用docker无法直接访问rclone mount之后的目录，大概是docker跟主机的网络不相通吧，Google一圈也没找到解决方案。但是神奇的是我用了`--allow-non-empty`居然就能访问了，我也不清楚原理，就是不知道加了这个flag会不会占用我服务器的空间  

另外马上就要断网了呜，下次能长时间用手机电脑什么的可能要等到高考结束了，难受。就酱叭  

![](/img/1.gif)

