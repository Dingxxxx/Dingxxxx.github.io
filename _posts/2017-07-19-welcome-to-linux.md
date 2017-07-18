---
layout: post
author: Ding
title:  "welcome to Linux"
date:   2017-07-19 
categories: Linux
tags: Linux Ubuntu
---

* content
{:toc}

Linux是一套免费使用和自由传播的类Unix操作系统，由Linus Torvalds开发，存在很多不同的发行版本，常见有Ubuntu、RedHat、Debian、CentOS。

其中Ubuntu是一个以桌面应用为主的开源GNU/Linux操作系统，既可以满足开发需求，又可以用于日常使用，所以选择安装最新的`Ubuntu 16.04`作为自己放弃Windows后的第一个操作系统。



## 安装Ubuntu 16.04

Linux中分区要以文件系统的方式挂载到系统中的挂载点上，就如同Windows中分区也要以Fat32或NTFS格式格式化成不成的盘符一样。至少 linux需要一个/分区（一定要打开启动选项，好像在安装过程中直接分区时不会提示，那就不用管了），一般也都会有个SWAP交换分区（这东西类似 Windows中的虚拟内存，但比那个还要专业，直接搞成一个分区形式了，而且Linux也有SWAP文件的形式出现）。

经过我的查询，单系统磁盘空间足够的情况下，比较好的分区方式是分为`/`根目录、`/home`用户工作目录和一个`SWAP`分区，其中/分区作为主分区，其他两个都作为逻辑分区，/分区大概10-100G用于存放系统文件，SWAP分区和内存一样大用作交换空间，剩余磁盘空间用作/home分区存放用户文件，方在重新安装系统时保留用户文件。

### 更新系统
安装完系统后用`sudo apt update`和`sudo apt upgrade`更新系统并升级软件。

### 安装显卡驱动
Ubuntu自带的Nouveau支持大多Nvidia显卡，但可以通过安装Nvidia专有驱动来获得更好的显卡支持，下载过程可能有点慢，安心等待即可。

![Nvidia driver]({{ site.url }}/_imgs/welcome_to_linux_1.png)


## 常用软件
