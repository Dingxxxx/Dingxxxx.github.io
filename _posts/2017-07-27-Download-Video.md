---
layout: post
author: Ding
title:  "用shell脚本批量下载视频"
date:   2017-07-27 
categories: Shell
tags: Shell Python
---

* content
{:toc}

>尝试编写的第一个shell小程序，可以用Github上python库you-get来批量下载bilibili网站上的视频。




```shell
    #!/bin/bash
    # create by Jerry Ding 2017.7.27
    # pip install you-get


    # 攻壳机动队
    mkdir 攻壳机动队
    cd 攻壳机动队
    var=28947
    while [ ${var} -le 28972 ]
    do
        you-get https://bangumi.bilibili.com/anime/1564/play#${var}
        var=$(($var + 1))
    done
    cd ..


    # 星际牛仔
    mkdir 星际牛仔
    cd 星际牛仔
    var=50510
    while [ ${var} -le 50535 ]
    do
        you-get https://bangumi.bilibili.com/anime/2261/play#${var}
        var=$(($var + 1))
    done
    cd ..


    # 虫师
    mkdir 虫师
    cd 虫师
    var=30872
    while [ ${var} -le 30897 ]
    do
        you-get https://bangumi.bilibili.com/anime/1715/play#${var}
        var=$(($var + 1))
    done
    cd ..

    # 命运石之门
    mkdir 命运石之门
    cd 命运石之门
    var=64190
    while [ ${var} -le 64212 ]
    do
        you-get https://bangumi.bilibili.com/anime/836/play#${var}
        var=$(($var + 1))
    done
    cd ..


    # 钢之炼金术师
    # https://bangumi.bilibili.com/anime/1088/play#19436
```



