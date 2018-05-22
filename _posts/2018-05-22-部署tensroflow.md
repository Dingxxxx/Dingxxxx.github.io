---
layout: post
author: Ding
title:  "部署Tensorflow-GPU版本"
date:   2018-05-22
categories: Linux
tags: tensoflow 
---

* content
{:toc}

> 在 Ubuntu 16:04 版本部署 Tensorflow-GPU




## 安装对应版本的GPU驱动

注意不同版本的CUDA所需要的驱动版本， CUDA 9.2 需要 NVIDIA-396.

`nvidia=smi`

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 396.24                 Driver Version: 396.24                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 850M    Off  | 00000000:01:00.0 Off |                  N/A |
| N/A   54C    P0    N/A /  N/A |    579MiB /  4046MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1920      G   /usr/lib/xorg/Xorg                           330MiB |
|    0      2963      G   compiz                                       148MiB |
|    0      5595      G   ...-token=80336B0E94BDC810846182985D133349    65MiB |
|    0      6530      G   ...-token=BC296FEDAE3F8E097753565A159A60D5    32MiB |
+-----------------------------------------------------------------------------+
```

## 安装 CUDA-9,2

从[官网](https://developer.nvidia.com/cuda-release-candidate-download)下载对应的runfile版本，注意不要用deb包装，会出问题。

```
sudo ./cuda_9.2.88_396.26_linux.run
```

```
NVIDIA Driver


Description

This package contains the operating system driver and
Do you accept the previously read EULA?
accept/decline/quit: accept

Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 396.26?
(y)es/(n)o/(q)uit: n

Install the CUDA 9.2 Toolkit?
(y)es/(n)o/(q)uit: y

Enter Toolkit Location
 [ default is /usr/local/cuda-9.2 ]: 

Do you want to install a symbolic link at /usr/local/cuda?
(y)es/(n)o/(q)uit: y

Install the CUDA 9.2 Samples?
(y)es/(n)o/(q)uit: y

Enter CUDA Samples Location
 [ default is /home/ding ]: 

Installing the CUDA Toolkit in /usr/local/cuda-9.2 ...
```

结果显示

```
===========
= Summary =
===========

Driver:   Not Selected
Toolkit:  Installed in /usr/local/cuda-9.2
Samples:  Installed in /home/ding, but missing recommended libraries

Please make sure that
 -   PATH includes /usr/local/cuda-9.2/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-9.2/lib64, or, add /usr/local/cuda-9.2/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run the uninstall script in /usr/local/cuda-9.2/bin

Please see CUDA_Installation_Guide_Linux.pdf in /usr/local/cuda-9.2/doc/pdf for detailed information on setting up CUDA.

***WARNING: Incomplete installation! This installation did not install the CUDA Driver. A driver of version at least 384.00 is required for CUDA 9.2 functionality to work.
To install the driver using this installer, run the following command, replacing <CudaInstaller> with the name of this run file:
    sudo <CudaInstaller>.run -silent -driver

Logfile is /tmp/cuda_install_25123.log

```

最后声明环境变量

```
export CUDA_HOME=/usr/local/cuda
export PATH=$PATH:$CUDA_HOME/bin
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

### 安装 CUDA 补丁包

```
sudo ./cuda_9.2.88.1_linux.run
```

```
NVIDIA Driver


Description

This package contains the operating system driver and
Do you accept the previously read EULA?
accept/decline/quit: accept

Enter CUDA Toolkit installation directory
 [ default is /usr/local/cuda-9.2 ]: 

Installation complete!
Installation directory: /usr/local/cuda-9.2
```

## 测试 CUDA

+ 查看 nvcc

```
nvcc -V

nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Wed_Apr_11_23:16:29_CDT_2018
Cuda compilation tools, release 9.2, V9.2.88
```

+ 测试　ｓａｍｐｌｅ
```
cd NVIDIA_CUDA-9.2_Samples/
make
cd cd bin/x86_64/linux/release/
./deviceQuery
```

```
 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GTX 850M"
  CUDA Driver Version / Runtime Version          9.2 / 9.2
  CUDA Capability Major/Minor version number:    5.0
  Total amount of global memory:                 4046 MBytes (4242604032 bytes)
  ( 5) Multiprocessors, (128) CUDA Cores/MP:     640 CUDA Cores
  GPU Max Clock rate:                            902 MHz (0.90 GHz)
  Memory Clock rate:                             900 Mhz
  Memory Bus Width:                              128-bit
  L2 Cache Size:                                 2097152 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(65536), 2D=(65536, 65536), 3D=(4096, 4096, 4096)
  Maximum Layered 1D Texture Size, (num) layers  1D=(16384), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(16384, 16384), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 1 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Compute Preemption:            No
  Supports Cooperative Kernel Launch:            No
  Supports MultiDevice Co-op Kernel Launch:      No
  Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 9.2, CUDA Runtime Version = 9.2, NumDevs = 1
Result = PASS
```

## 安装cudnn

安装 cuDNN 比较简单，解压后把相应的文件拷贝到对应的 CUDA 目录下即可：

```
sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/
sudo chmod a+r /usr/local/cuda/include/cudnn.h
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
```

## 安装 Tensorflow-GPU

+ 创建Tensorflow环境

```
conda create -n tensorflow python=3.6
```

+ 安装

```
conda install tensorflow-gpu
```

+ 测试

```python
import tensorflow as tf
import numpy as np
x = tf.placeholder("float",shape=[None,1])


W = tf.Variable(tf.zeros([1,1]))
b = tf.Variable(tf.zeros([1]))


y = tf.matmul(x,W) +b

y_ = tf.placeholder("float",[None,1])

cost = tf.reduce_sum(tf.pow((y_-y),2))

train_step = tf.train.GradientDescentOptimizer(0.001).minimize(cost)

init = tf.initialize_all_variables()

sess = tf.Session()
sess.run(init)


All_x = np.empty(shape=[1,1])
All_y = np.empty(shape=[1,1])


for i in range(1000):
    x_s = np.random.rand(1,1)

    y_s = np.dot([[0.33]],np.random.rand(1,1)) + 0.33

    feed = {x: x_s, y_: y_s}
    sess.run(train_step,feed_dict=feed)
    print("After %d iteration:"%i)
    print("W : %f"%sess.run(W))
    print("b : %f"%sess.run(b))

    All_x = np.concatenate((All_x,x_s))
    All_y = np.concatenate((All_y,y_s))

print(All_x)
print(All_y)
```


```
WARNING:tensorflow:From /home/ding/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow/python/util/tf_should_use.py:118: initialize_all_variables (from tensorflow.python.ops.variables) is deprecated and will be removed after 2017-03-02.
Instructions for updating:
Use `tf.global_variables_initializer` instead.
2018-05-22 11:21:19.842940: I tensorflow/core/platform/cpu_feature_guard.cc:140] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2018-05-22 11:21:19.901902: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:898] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
2018-05-22 11:21:19.902359: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1344] Found device 0 with properties: 
name: GeForce GTX 850M major: 5 minor: 0 memoryClockRate(GHz): 0.9015
pciBusID: 0000:01:00.0
totalMemory: 3.95GiB freeMemory: 3.25GiB
2018-05-22 11:21:19.902373: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1423] Adding visible gpu devices: 0
2018-05-22 11:21:20.424261: I tensorflow/core/common_runtime/gpu/gpu_device.cc:911] Device interconnect StreamExecutor with strength 1 edge matrix:
2018-05-22 11:21:20.424285: I tensorflow/core/common_runtime/gpu/gpu_device.cc:917]      0 
2018-05-22 11:21:20.424290: I tensorflow/core/common_runtime/gpu/gpu_device.cc:930] 0:   N 
2018-05-22 11:21:20.424468: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1041] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 2974 MB memory) -> physical GPU (device: 0, name: GeForce GTX 850M, pci bus id: 0000:01:00.0, compute capability: 5.0)
```