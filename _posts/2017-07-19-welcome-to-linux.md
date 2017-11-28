---
layout: post
author: Ding
title:  "Welcome to Linux"
date:   2017-07-19
categories: Linux
tags: Linux Ubuntu
---

* content
{:toc}

> Linux是一套免费使用和自由传播的类Unix操作系统，由Linus Torvalds开发，存在很多不同的发行版本，常见有Ubuntu、RedHat、Debian、CentOS。
>
> 其中Ubuntu是一个以桌面应用为主的开源GNU/Linux操作系统，既可以满足开发需求，又可以用于日常使用，所以我选择安装>最新的`Ubuntu 16.04`作为自己放弃Windows后的第一个操作系统，本文记录了安装部署Ubuntu环境的过程。



## 安装Ubuntu 16.04

Linux中分区要以文件系统的方式挂载到系统中的挂载点上，就如同Windows中分区也要以Fat32或NTFS格式格式化成不成的盘符一样。至少 linux需要一个/分区（一定要打开启动选项，好像在安装过程中直接分区时不会提示，那就不用管了），一般也都会有个SWAP交换分区（这东西类似 Windows中的虚拟内存，但比那个还要专业，直接搞成一个分区形式了，而且Linux也有SWAP文件的形式出现）。

经过我的查询，单系统磁盘空间足够的情况下，比较好的分区方式是分为`/`根目录、`/home`用户工作目录和一个`SWAP`分区，其中/分区作为主分区，其他两个都作为逻辑分区，/分区大概10-100G用于存放系统文件，SWAP分区和内存一样大用作交换空间，剩余磁盘空间用作/home分区存放用户文件，方在重新安装系统时保留用户文件。

### 更新系统
安装完系统后用`sudo apt update`和`sudo apt upgrade`更新系统并升级软件。

### 安装显卡驱动
Ubuntu自带的Nouveau支持大多Nvidia显卡，但可以通过安装Nvidia专有驱动来获得更好的显卡支持，下载过程可能有点慢，安心等待即可。

![Nvidia driver](/images/welcome_to_linux/1.png)

## Ubuntuan安装软件的方式

### Debian Packager(dpkg)

dpkg是Debian软件包管理器的基础,本身是一个底层的工具,不能远程下载包以及处理包的依赖关系。

- 安装软件

        dpkg -i <file>.deb

- 移除软件

        dpkg -r <file>.deb

- 移除软件及配置文件

        dpkg -P <file>.deb


### Advanced Packaging Tool(APT)

apt工具可以通过`/etc/apt/source.list`从远程源服务器下载程序并自行安装依赖关系。

- 更新源

        sudo apt-get update

- 更新已安装软件

        sudo apt-get upgrade

- 升级系统

        sudo apt-get dist-upgrade

- 安装软件

        sudo apt-get install <package>

- 缺失依赖时修复

        sudo apt-get -f install

- 移除软件

        sudo apt-get remove <package>

- 移除软件和配置文件

        sudo apt-get remove --purge <package>

- 移除软件及其依赖和配置文件

        sudo apt-get autoremove --purge <package>

### Synaptic Package Manager

新立得软件包管理器是dpkg命令的图形化前端，能够在图形界面内完成LINUX系统软件的搜寻、安装和删除，相当于终端里的apt命令。使用新立得软件包管理器的同时不能使用终端安装软件，因为它们实质上是一样的。

- 安装新立得软件包管理器

        sudo apt-get install synaptic



## 安装常见软件

- 安装受限制的解码器
    包括解码器、flashplayer、java虚拟机、微软的Truetype字体等受限代码。

        sudo apt-get install ubuntu-restricted-extras

- 搜狗输入法

        sudo apt-get install sogoupinyin

- JDK
    JRE是运行Java需要的最基本的环境包，JDK是进行Java开发所需要的完整工具包，有开源的OpenJDK和官方的OracleJDK两个版本，选择安装OpenJDK。

        sudo apt install openjdk-9-jdk

- Chrome

        sudo apt-get install google-chrome-stable

- 安装TLP管理笔记本电池寿命

        sudo add-apt-repository ppa:linrunner/tlp
        sudo apt-get update
        sudo apt-get install tlp tlp-rdw
        sudo tlp start

- BleachBit 系统清理工具

       sudo apt-get install bleachbit

- Unity Tweak Tool

        sudo apt-get install unity-tweak-tool

- Flatabulous主题及Ultra-flat图标
- Docky
    类似于Mac下的docky，实际使用中并不符合我的审美，已删除。

        sudo apt install docky

- 系统监视器System Monitor Indicator及配置

        sudo apt-add-repository ppa:alexeftimie/ppa
        sudo apt-get update
        sudo apt-get install indicator-sysmonitor

- 中文字体
- touchpad-indicator
    笔记本触摸板太过灵敏，在使用键盘时经常勿碰，可以用这个插件方便的开关触摸板，并设置插入鼠标时自动禁用触摸板。

        sudo add-apt-repository ppa:lorenzo-carbonell/atareao
        sudo apt-get update
        sudo apt-get install touchpad-indicator

- Latex
    参考自http://blog.csdn.net/miaoqiucheng/article/details/53082326。
- GIMP

        sudo apt-get install gimp

- VLC媒体播放器

        sudo apt-get install vlc

- WPS Office
- StarDict
- Shutter
    截图工具，快捷键为`Ctrl+Alt+A`

        sudo apt-get install shutter

- steam

        sudo apt-get install steam

- 网易云播放器

        sudo apt install netease-cloud-music

- 微信
- Xmind
- Oracle Virtual Box
- Okular
    一款PDF阅读器
- Mendelay
    文献管理工具

- xpad
    便签

- uget + aria2
    类似于迅雷的下载工具

- filezilla
    ftp下载工具


## 科学上网

### 蓝灯 Lantern

[蓝灯](https://github.com/getlantern)官方GitHub可以获取直接获取最新版本，在Ubuntu系统下存在使用全局代理与网易云音乐冲突的问题，暂时没有发现在不使用全局代理的情况下使用Lantern。

    sudo apt-get install lantern


### 影梭 Shadowsocks

shadowsocks是一种基于Socks5代理方式的网络数据加密传输包，并采用Apache许可证、GPL、MIT许可证等多种自由软件许可协议开放源代码。shadowsocks分为服务器端和客户端，在使用之前，需要先将服务器端部署到服务器上面，然后通过客户端连接并创建本地代理。

- 服务器端

    使用网上别人提供的服务器。

- 客户端

    [ss-qt5](https://github.com/shadowsocks/shadowsocks-qt5)是GitHub上一个跨平台的开源shadowsocks GUI客户端，可以通过apt安装。

        sudo apt-get install shadowsocks-qt5

    配置好连接并测试延迟之后，打开`系统设置-网络-网络代理`设置代理，有手动和自动代理两种方式。

    * 手动代理即全局代理，使用这种方式在没有连接shadowsocks服务器的情况下不能使用国内网站，比较浪费shadowsocks流量。
    * 自动代理通过Genpac生成的pac文件自定义代理规则，需要科学上网时连接shadowsocks服务器即可，正常上网无需连接shadowsocks服务器。

## 编程环境

### 安装git 并配置

        git config --global user.name "YOUR NAME"
        git config --global user.email "YOUR EMAIL ADDRESS"

### Anaconda
前往[官网](https://www.continuum.io/downloads)选择相应的系统版本和python版本，下载Anaconda安装文件。
运行下载得到的sh文件进入安装程序。

        bash Anaconda3-4.4.0-Linux-x86_64.sh

安装完成后在shell中输入`python`得到如下显示即安装成功。
![anaconda](/images/welcome_to_linux/2.png)

### Sublime Text并配置中文

### 安装开发环境PyCharm和IntelliJ

添加sublime text3软件源并安装

	sudo add-apt-repository ppa:webupd8team/sublime-text-3
	sudo apt-get update
	sudo apt-get install sublime-text-installer

不能输入中文的问题，GitHub上的项目[sublime-text-imfix](https://github.com/lyfeyaj/sublime-text-imfix)可以解决，仅支持在Terminal中输入subl调用Sublime Text时输入中文。

    sudo apt-get update && sudo apt-get upgrade
    git clone https://github.com/lyfeyaj/sublime-text-imfix.git
    cd sublime-text-imfix
    ./sublime-imfix

### 安装Visual Studio Code

### 安装postgresql和pgAdmin

### 安装CUDA和CUdnn

### 安装Tensorflow
