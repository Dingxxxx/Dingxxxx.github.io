---
layout: post
author: Ding
title:  "Cartopy Document--参考坐标系CRS"
date:   2017-09-13
categories: python cartopy
tags: python cartopy matplotlib visualising
---

* content
{:toc}

> Starting in 2016, Basemap came under new management. The Cartopy project will replace Basemap, but it hasn’t yet implemented all of Basemap’s features. All new software development should try to use Cartopy whenever > possible, and existing software should start the process of switching over to use Cartopy. All maintenance and development efforts should be focused on Cartopy.
>
> 这是cartopy 的简略中文文档






## 安装cartopy

直接使用conda安装cartopy及相关库.

```shell
conda install -c conda-forge cartopy

```

国内conda-forge源连接不稳定, 使用清华大学的镜像.

```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda install -c cartopy
```

## Coordinate reference systems in Cartopy  参考坐标系CRS

### 三种坐标系

+ 空间坐标系（3维坐标系，同于数学的）

一般是指纯粹的三维空间的坐标系，表示方式大致有x、y、z坐标和a、b、r极坐标，就是数学里的概念；

+ 地理坐标系（3维的球面坐标系，与地球表面切合的）

既然有地理两个字肯定与地理的东西有关，首先要知道地理坐标系的目的（就是为了定位好在地球表面的城市等地理要素的位置），它的形式与空间一样，，但由于地球表面高度不同、受重力场影响不同、受海洋与陆地的不同性质的影响，在地球里面的哪个位置才是只是要处理一下哪里是球心、原点、半径有多大。要定位一个位置不难，但要对一个地区定位就有点难了（因为到时候再投影到二维平面上时会产生不同程度的误差，我们要尽量减少这个误差）。通常一个地理坐标系需要的要素有：对地球表面或对某部分地区国家的地表进行模拟的实际地球的代替品——椭球体、要有统一基准作为高度起算点的基准面，也正因为不同类型的椭球体（长轴、短轴、扁率不同而异）对每个国家或地区的地表拟合效果不同（也就造成模拟效果与投影为平面时误差的不同和与实际重力磁场等实际要素的不同），可能这个椭球体对于这个国家的实际状况模拟很贴切，而对于另一些国家却会有很大误差。

+ 投影坐标系（是将3维坐标按照特定方式投影后的2维坐标系）

同样的确定好椭球体后还要投影到二维平面才能方便我们日常观看和分析，这时怎样的投影方式才能更好在平面上反映实际的地面状况呢？与确定椭球体一样——不同投影方式对不同地区会有不同的反映效果，所以哪个最好，说不准，也就造成会有这么多的投影了

`cartopy.crs.CRS`是cartopy中所有坐标参考系的父类, 即所有的投影都可以用以下列出的方式交互.


```
class cartopy.crs.CRS(proj4_params, globe=None)
    通过proj.4坐标投影库创建一个坐标参考系.

    proj4_params:一个可迭代的键值对。
        通过proj4参数来定义CRS, 参数将会覆盖Globe参数中的参数.

    globe:一个可选的Globe实例.
#####################################################################
    golble:CRS的Globe实例.

    as_geocentric():返回一个新的与该CRS相同的地心参考系.

    as_geodetic():返回一个新的与该CRS相同的大地测量参考系.

    transform_point(x, y, src_crs)
        将给定的float64坐标, 从src_crs参考系转化为该参考系，返回(x,y)

    transform_points(src_crs, x, y[, z])
        对于给定的一组x,y[,z]从src_crs转到该坐标系.

    transform_vectors(src_crs, x, y, u, v)
        对于src_crs参考系中的向量, (x, y)为向量在src_crs中的坐标, (u, v)为向量东方向和北方向的向量分量。
        返回(ut, vt)转化后的向量分量.
```

`carto.crs.Golbe`用来封装cartopy坐标系中底层的球体和椭球. 每一个坐标系都有一个相关联的Globe对象, 通常他就是代表参考椭球的默认Globe(如wgs84).


```
class cartopy.crs.Globe(datum=None, ellipse='WGS84', semimajor_axis=None,
                        semiminor_axis=None, flattening=None,
                        inverse_flattening=None, towgs84=None)
    定义了一个椭球体和它怎样映射到现实世界.

    关键字:
        datum: Proj4中的datum,默认为no datum.
        ellipse: Proj4中的ellps, 默认为'WGS84'.
        semimajor_axis: 椭球的长半轴.
        semiminor_axis: 椭球的短半轴.
        flattening: 椭球的扁率.
        inverse_flattening: 反扁率.
        towgs84: 通过Proj4定义.
        nadgrids: 通过Proj4定义.
```

`cartopy.crs.Projection`表示了一个二维的坐标系,可以直接当做地图画在一张扁平的纸上. `Projection`是Cartopy中所有投影的父类.


```
class cartopy.crs.Projection
    定义了一个具有平坦拓扑关系和欧几里得距离的投影坐标系.

    proj4_params:一个可迭代的键值对。
    通过proj4参数来定义CRS, 参数将会覆盖Globe参数中的参数.

    globe:一个可选的Globe实例.

#####################################################################
    project_geometry(geometry, src_crs=None)
        该函数将一个几何体投影到该投影上.

        geometry: 待投影的几何体.
        src_crs: geometry的坐标系,如果为None则认为是大地测量的CRS.

        返回一个变形后的几何体.
```
