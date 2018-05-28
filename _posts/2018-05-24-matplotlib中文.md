---
layout: post
author: Ding
title:  "matplotlib中文配置"
date:   2018-05-28
categories: Python
tags: Python
---

* content
{:toc}

> 中文配置





## 找到matplotlib目录

```python
import matplotlib as mpl
print(mpl.matplotlib_fname())
print(mpl.rcParams['font.serif'])
print(mpl.rcParams['font.sans-serif'])
```

得到
`/home/ding/anaconda3/envs/tensorflow/lib/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc`

## 删除缓存

```shell
rm -rf /home/ding/.cache/matplotlib/
```

## 拷贝字体文件

```shell
fc-list :lang=zh|grep 中文
sudo cp /usr/share/fonts/中文/msyh.ttf /home/ding/anaconda3/envs/tensorflow/lib/python3.6/site-packages/matplotlib/mpl-data/fonts/ttf
```

## 修改配置文件

```
gedit /home/ding/anaconda3/envs/tensorflow/lib/python3.6/site-packages/matplotlib/mpl-data/matplotlibrc
```

修改

```
font.family         : sans-serif
font.sans-serif     : Microsoft YaHei, ...
axes.unicode_minus  : false 
```


## 测试

```python
#coding:utf-8
import matplotlib.pyplot as plt
plt.rcParams['font.sans-serif']=['SimHei'] #用来正常显示中文标签
plt.rcParams['axes.unicode_minus']=False #用来正常显示负号
plt.plot((1,2,3),(4,3,-1))
plt.xlabel(u'横坐标')
plt.ylabel(u'纵坐标')
plt.show()
```