---
layout: post
author: Ding
title: 安装Python版本的opencv
date: 2017-11-08
categories: opencv
tags:
- 机器学习
- opencv
- python
---

* content
{:toc}

> opencv是一个很好的图像处理库，主要由c和少量c++写成，具有python接口，是机器学习中处理图像的主要模块。









## 安装opencv并配置环境

可以参考(http://www.jianshu.com/p/67293b547261)

### 通过apt安装opencv

在Linux中安装OpenCV是非常方便的，大多数Linux的发行版都支持包管理器的安装，比如在Ubuntu 16.04 LTS中，只需要在终端中输入：

```
sudo apt install libopencv-dev python-opencv
```

### 通过源码编译opencv
当然也可以通过[官网](https://opencv.org/)下载最新版本的opencv源码编译安装，第一步先安装各种依赖：

参考这个官网给出的[安装步骤](https://docs.opencv.org/3.3.1/dd/dd5/tutorial_py_setup_in_fedora.html)

```
sudo apt install build-essential

sudo apt install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev

sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```

进入下载好的opencv文件夹：

```
mkdir release

cd release

cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
```
结果IPPICV报错。

#### 自行下载第三方库

可以到[Github](https://github.com/opencv/opencv_3rdparty/branches/all)上找到出错的ippicv(并行计算库)，下载下来。

下载文件的地方位置在.cache/ippicv下，带md5的文件名字，需要把ippicv_2017u3_lnx_intel64_general_20170822.tgz前面加上md5

找到提示对应的文件，将文件拷贝到opencv-3.3.1同级目录
执行下面命令

```
ipp_file=ippicv_2017u3_lnx_intel64_general_20170822.tgz &&ipp_hash=$(md5sum ../$ipp_file | cut -d" " -f1) &&ipp_dir=.cache/ippicv                           &&mkdir -p $ipp_dir &&cp ../$ipp_file $ipp_dir/$ipp_hash-$ipp_file
在.cache/ippicv文件夹下有4e0352ce96473837b1d671ce87f17359-ippicv_2017u3_lnx_intel64_general_20170822.tgz文件
```

放好ippicv后可以重新cmake

#### 忽略 IPPICV


编译并安装

```
make
sudo make install
```



### 通过Anaconda安装

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/menpo/
conda install -c https://conda.binstar.org/menpo opencv
```
## 测试

```python
import cv2
print(cv2.__version__)
```



### 基本使用

[知乎](https://zhuanlan.zhihu.com/p/24425116)

[教程](https://segmentfault.com/a/1190000003742422)
