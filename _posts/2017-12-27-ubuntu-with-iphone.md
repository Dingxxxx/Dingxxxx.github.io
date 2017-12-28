---
layout: post
author: Ding
title: 在Ubuntu上搞定iPhone
date: 2017-12-27
categories: Linux
tags:
- Ubuntu
- iPhone
---

* content
{:toc}

> 参考[博客](http://blog.csdn.net/fengzei886/article/details/53380009)






## 下载安装包


```
mkdir libimobiledevice
cd libimobiledevice

git clone https://github.com/libimobiledevice/libplist.git
git clone https://github.com/libimobiledevice/usbmuxd.git
git clone https://github.com/libimobiledevice/libimobiledevice.git
git clone https://github.com/libimobiledevice/libusbmuxd.git
git clone https://github.com/libimobiledevice/ideviceinstaller.git
git clone https://github.com/libimobiledevice/ifuse.git
git clone https://github.com/libimobiledevice/libirecovery.git
git clone https://github.com/libimobiledevice/libideviceactivation.git
```


## 开始安装

+ 安装依赖

```
sudo apt-get install make automake autoconf libtool pkg-config
sudo apt-get install python-dev
```
+ 安装libplist

```
cd libplist
./autogen.sh
make
install
```

这个时候出现了错误：无法连接到python库。原因是Anaconda改变了python的位置。解决方法：

```
which python
```

看一下Anaconda把python装在那里，用一个软链接把系统中python的默认位置链接到该位置上。

```
ln -s /usr/bin/python /home/ding/anaconda3/bin/python
```

此时可以完成

```
./autogen.sh
make
```

make时再次报错，检查`Makefile`因为python2没有装相应的cpython，安装它

```
pip install cython
```

然后

```
sudo make
sudo make install
```

+ 安装 libusbmuxd

```
cd ../libusbmuxd
vim README
./autogen.sh
sudo make
sudo make install
```

+ 安装 libimobiledevice:

```
cd ../libimobiledevice
vim README
sudo apt-get install libssl-dev
./autogen.sh
sudo make && make install
```

+ 安装 usbmuxd:

先安装 libsub

```
cd ../usbmuxd
vim README
sudo apt-get intall libusb-1.0-0-dev
./autogen.sh
sudo make
sudo make install
```

+ 安装 ifuse:

```
cd ../ifuse/
vim README
sudo apt-get install libfuse-dev
./autogen.sh
sudo make && make install
```


派对手机

```
ding@ding:~$ idevicepair pair
SUCCESS: Paired with device 9a8110fc972f8f1dce4bec4f786ad1a6316dbea7
```
