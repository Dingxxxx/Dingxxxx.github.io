---
layout: post
author: Ding
title:  "Cartopy Document--通过matplotlib绘制地图"
date:   2017-09-13
categories: Python
tags: 
- Python 
- cartopy 
- matplotlib 
- visualising
---

* content
{:toc}

> Starting in 2016, Basemap came under new management. The Cartopy project will replace Basemap, but it hasn’t yet implemented all of Basemap’s features. All new software development should try to use Cartopy whenever > possible, and existing software should start the process of switching over to use Cartopy. All maintenance and development efforts should be focused on Cartopy.
>
> 这是cartopy 的简略中文文档









## Using cartopy with matplotlib

### 定义地图
cartopy提供了可以通过matplotlib进行地图绘制的简单接口. 创建一个基本的地图只需要告诉matplotlib使用一个具体的地图投影, 然后在axes对象上加一些海岸线(coastline).


```python
import cartopy.crs as ccrs
import matplotlib.pyplot as plt

ax = plt.axes(projection=ccrs.PlateCarree())
ax.coastlines()

plt.show()
```


![png](/images/Cartopy/output_64_0.png)


`plt.axes(projection=ccrs.PlateCarree()`设置了一个GeoAxes实例, GeoAxes有很多与地图相关的方法, 上面的例子中我们用`coastlines()`来向地图中加海岸线.

我们可以用另一个投影创建一幅地图, 并用`stock_img()`f方法来给地图添加底图.


```python
import cartopy.crs as ccrs
import matplotlib.pyplot as plt

ax = plt.axes(projection=ccrs.Mollweide())
ax.stock_img()
plt.show()
```


![png](/images/Cartopy/output_66_0.png)


### 添加数据
一旦有了你想要的地图, 数据可以用和正常的matplotlib axes对象完全相同的方式添加进来. 默认情况下, 数据使用的坐标系和GeoAxes本身的坐标系是相同的. 你可以通过carto.crs.CRS实例中transform进行合适的变换来控制数据的坐标系.


```python
import cartopy.crs as ccrs
import matplotlib.pyplot as plt

ax = plt.axes(projection=ccrs.PlateCarree())
ax.stock_img()

ny_lon, ny_lat = -75, 43
delhi_lon, delhi_lat = 77.23, 28.61

# 大地坐标系
plt.plot([ny_lon, delhi_lon], [ny_lat, delhi_lat],
         color='blue', linewidth=2, marker='o',
         transform=ccrs.Geodetic(),
         )

# 和地图相同的坐标系
plt.plot([ny_lon, delhi_lon], [ny_lat, delhi_lat],
         color='gray', linestyle='--',
         transform=ccrs.PlateCarree(),
         )

plt.text(ny_lon - 3, ny_lat - 12, 'New York',
         horizontalalignment='right',
         transform=ccrs.Geodetic())

plt.text(delhi_lon + 3, delhi_lat - 12, 'Delhi',
         horizontalalignment='left',
         transform=ccrs.Geodetic())

plt.show()
```


![png](/images/Cartopy/output_68_0.png)


注意到在纽约和新德里之间的蓝色线在扁平的PlateCarree地图上不是一条直线,因为大地坐标系是一个完全的球面坐标系, 其两点间的一条线被定义为在球面上的最短路径而不是在笛卡尔空间中.

Note:

By default, matplotlib automatically sets the limits of your Axes based on the data that you plot. Because cartopy implements a GeoAxes class, this equates to the limits of the resulting map. Sometimes this autoscaling is a desirable feature and other times it is not.

To set the extens of a cartopy GeoAxes, there are several convenient options:
+ For “global” plots, use the set_global() method.

+ To set the extents of the map based on a bounding box, in any coordinate system, use the set_extent() method.

+ Alternatively, the standard limit setting methods can be used in the GeoAxes’s native coordinate system (e.g. set_xlim() and set_ylim()).

## More advanced mapping with cartopy and matplotlib
从一开始,cartopy的目的就是简化和提高科学数据地图可视化的质量. 由于cartopy交互的简易,很多情况进行这样的可视化最困难的是获取数据. 为了解决这个问题,一个python扩展包 Iris 可以用来加载和保存来自各种各样网格数据集的数据. 下面的例子有些使用Iris加载数据,有些用netCDF4的python扩展包,来展示一些列不同的数据获取方法.

### 等高线 plt.contourf()


```python
import os
import matplotlib.pyplot as plt
from netCDF4 import Dataset as netcdf_dataset
import numpy as np

from cartopy import config
import cartopy.crs as ccrs


# get the path of the file. It can be found in the repo data directory.
fname = os.path.join(config["repo_data_dir"],
                     'netcdf', 'HadISST1_SST_update.nc'
                     )

dataset = netcdf_dataset(fname)
sst = dataset.variables['sst'][0, :, :]
lats = dataset.variables['lat'][:]
lons = dataset.variables['lon'][:]

ax = plt.axes(projection=ccrs.PlateCarree())

plt.contourf(lons, lats, sst, 60,
             transform=ccrs.PlateCarree())

ax.coastlines()

plt.show()
```


![png](/images/Cartopy/output_71_0.png)


### 区块 plt.pcolormesh()


```python
import iris
import matplotlib.pyplot as plt

import cartopy.crs as ccrs


# load some sample iris data
fname = iris.sample_data_path('rotated_pole.nc')
temperature = iris.load_cube(fname)

# iris comes complete with a method to put bounds on a simple point
# coordinate. This is very useful...
temperature.coord('grid_latitude').guess_bounds()
temperature.coord('grid_longitude').guess_bounds()

# turn the iris Cube data structure into numpy arrays
gridlons = temperature.coord('grid_longitude').contiguous_bounds()
gridlats = temperature.coord('grid_latitude').contiguous_bounds()
temperature = temperature.data

# set up a map
ax = plt.axes(projection=ccrs.PlateCarree())

# define the coordinate system that the grid lons and grid lats are on
rotated_pole = ccrs.RotatedPole(pole_longitude=177.5, pole_latitude=37.5)
plt.pcolormesh(gridlons, gridlats, temperature, transform=rotated_pole)

ax.coastlines()

plt.show()
```

![png](/images/Cartopy/output_73_0.png)


### 导入图像 ax.imshow()

```python
import os
import matplotlib.pyplot as plt

from cartopy import config
import cartopy.crs as ccrs


fig = plt.figure(figsize=(8, 12))

# get the path of the file. It can be found in the repo data directory.
fname = os.path.join(config["repo_data_dir"],
                     'raster', 'sample', 'Miriam.A2012270.2050.2km.jpg'
                     )
img_extent = (-120.67660000000001, -106.32104523100001, 13.2301484511245, 30.766899999999502)
img = plt.imread(fname)

ax = plt.axes(projection=ccrs.PlateCarree())
plt.title('Hurricane Miriam from the Aqua/MODIS satellite\n'
          '2012 09/26/2012 20:50 UTC')

# set a margin around the data
ax.set_xmargin(0.05)
ax.set_ymargin(0.10)

# add the image. Because this image was a tif, the "origin" of the image is in the
# upper left corner
ax.imshow(img, origin='upper', extent=img_extent, transform=ccrs.PlateCarree())
ax.coastlines(resolution='50m', color='black', linewidth=1)

# mark a known place to help us geo-locate ourselves
ax.plot(-117.1625, 32.715, 'bo', markersize=7, transform=ccrs.Geodetic())
ax.text(-117, 33, 'San Diego', transform=ccrs.Geodetic())

plt.show()
```


![png](/images/Cartopy/output_75_0.png)


### 矢量场
cartopy有强大的矢量场绘图功能. 有三种不同的矢量场可视化选项: quivers, barbs, streamplots, 每种有对应不同矢量场形式的适用场合.

+ arrows


```python
import matplotlib.pyplot as plt
import numpy as np

import cartopy
import cartopy.crs as ccrs


def sample_data(shape=(20, 30)):
    """
    Returns ``(x, y, u, v, crs)`` of some vector data
    computed mathematically. The returned crs will be a rotated
    pole CRS, meaning that the vectors will be unevenly spaced in
    regular PlateCarree space.

    """
    crs = ccrs.RotatedPole(pole_longitude=177.5, pole_latitude=37.5)

    x = np.linspace(311.9, 391.1, shape[1])
    y = np.linspace(-23.6, 24.8, shape[0])

    x2d, y2d = np.meshgrid(x, y)
    u = 10 * (2 * np.cos(2 * np.deg2rad(x2d) + 3 * np.deg2rad(y2d + 30)) ** 2)
    v = 20 * np.cos(6 * np.deg2rad(x2d))

    return x, y, u, v, crs


def main():
    fig = plt.figure(figsize=(8, 8))
    ax = plt.axes(projection=ccrs.Orthographic(-10, 45))

    ax.add_feature(cartopy.feature.OCEAN, zorder=0)
    ax.add_feature(cartopy.feature.LAND, zorder=0, edgecolor='black')

    ax.set_global()
    ax.gridlines()

    x, y, u, v, vector_crs = sample_data()
    ax.quiver(x, y, u, v, transform=vector_crs)

    plt.show()


if __name__ == '__main__':
    main()
```


![png](/images/Cartopy/output_77_0.png)


+ barbs


```python
import matplotlib.pyplot as plt

import cartopy.crs as ccrs
from cartopy.examples.arrows import sample_data


def main():
    ax = plt.axes(projection=ccrs.PlateCarree())
    ax.set_extent([-90, 75, 10, 60])
    ax.stock_img()
    ax.coastlines()

    x, y, u, v, vector_crs = sample_data(shape=(10, 14))
    ax.barbs(x, y, u, v, length=5,
             sizes=dict(emptybarb=0.25, spacing=0.2, height=0.5),
             linewidth=0.95, transform=vector_crs)

    plt.show()


if __name__ == '__main__':
    main()
```


![png](/images/Cartopy/output_79_0.png)


+ streamplot


```python
import matplotlib.pyplot as plt

import cartopy.crs as ccrs
from cartopy.examples.arrows import sample_data


def main():
    ax = plt.axes(projection=ccrs.PlateCarree())
    ax.set_extent([-90, 75, 10, 60])
    ax.coastlines()

    x, y, u, v, vector_crs = sample_data(shape=(80, 100))
    magnitude = (u ** 2 + v ** 2) ** 0.5
    ax.streamplot(x, y, u, v, transform=vector_crs,
                  linewidth=2, density=2, color=magnitude)
    plt.show()


if __name__ == '__main__':
    main()
```


![png](/images/Cartopy/output_81_0.png)


Since both quiver() and barbs() are visualisations which draw every vector supplied, there is an additional option to “regrid” the vector field into a regular grid on the target projection (done via cartopy.vector_transform.vector_scalar_to_grid()). This is enabled with the regrid_shape keyword and can have a massive impact on the effectiveness of the visualisation:


```python
"""
Regridding vectors with quiver
------------------------------

This example demonstrates the regridding functionality in quiver (there exists
equivalent functionality in :meth:`cartopy.mpl.geoaxes.GeoAxes.barbs`).

Regridding can be an effective way of visualising a vector field, particularly
if the data is dense or warped.
"""
import matplotlib.pyplot as plt
import numpy as np

import cartopy.crs as ccrs


def sample_data(shape=(20, 30)):
    """
    Returns ``(x, y, u, v, crs)`` of some vector data
    computed mathematically. The returned CRS will be a North Polar
    Stereographic projection, meaning that the vectors will be unevenly
    spaced in a PlateCarree projection.

    """
    crs = ccrs.NorthPolarStereo()
    scale = 1e7
    x = np.linspace(-scale, scale, shape[1])
    y = np.linspace(-scale, scale, shape[0])

    x2d, y2d = np.meshgrid(x, y)
    u = 10 * np.cos(2 * x2d / scale + 3 * y2d / scale)
    v = 20 * np.cos(6 * x2d / scale)

    return x, y, u, v, crs


def main():
    plt.figure(figsize=(8, 10))

    x, y, u, v, vector_crs = sample_data(shape=(50, 50))
    ax1 = plt.subplot(2, 1, 1, projection=ccrs.PlateCarree())
    ax1.coastlines('50m')
    ax1.set_extent([-45, 55, 20, 80], ccrs.PlateCarree())
    ax1.quiver(x, y, u, v, transform=vector_crs)

    ax2 = plt.subplot(2, 1, 2, projection=ccrs.PlateCarree())
    plt.title('The same vector field regridded')
    ax2.coastlines('50m')
    ax2.set_extent([-45, 55, 20, 80], ccrs.PlateCarree())
    ax2.quiver(x, y, u, v, transform=vector_crs, regrid_shape=20)

    plt.show()


if __name__ == '__main__':
    main()
```


![png](/images/Cartopy/output_83_0.png)


## cartopy与matplotlib的接口
将cartopy整合到matplotlib中主要靠的就是GeoAxes类, 它是通用matplotlib Axes的子类. GeoAxes类在Axes类的基础上加了一些专门用来绘制地图的函数. 大多数从原始的Axes类中获得的方法都改进了预期的效果, 但还有一些由于标准的matplotlib Axes用笛卡尔坐标系来处理数据而收到了限制(大多数这些问题已经或者将会提交给matplotlib项目).

###  GeoAxs对象
1. **class cartopy.mpl.geoaxes.GeoAxes**(\*args, \*\*kwargs)

  matplotlib.axes.Axes的子类,代表了一个地图投影.

  当用projecttion关键字创建改类后, 可以用该类取代matplotlib中的Axes类.


```python
# Set up a standard map for latlon data.
geo_axes = pyplot.axes(projection=cartopy.crs.PlateCarree())

# Set up an OSGB map.
geo_axes = pyplot.subplot(2, 2, 1, projection=cartopy.crs.OSGB())
```

通过transform关键字向它的plot方法传递一个源投影, 可以将标准的matplotlib plot结果从源坐标系转化到目标投影.


```python
# Plot latlon data on an OSGB map.
pyplot.axes(projection=cartopy.crs.OSGB())
pyplot.contourf(x, y, data, transform=cartopy.crs.PlateCarree())
```

创建GeoAxes对象需要标准matplotlib Axes的参数和一个map_projection参数, map_projection参数表示这个GeoAxes的目标投影, 其余的参数直接传给matplotlib.axes.Axes.

方法:

+ **add_feature**(feature, \*\*kwargs)

    向axes中添加Feather实例.

    feature: 一个Feature实例

    kwargs: 用来控制绘制feature的参数, 标准的matplotlib关键字如'facecolor','alpha'等.

    返回: cartopy.mpl.feature_artist.FeatureArtist 实例用于绘制feature.


+ **add_geometries**(geoms, crs, \*\*kwargs)   

    将一个给定crs的几何形状加入axes中.

    geoms: 一个几何体的集合.

    crs: geoms的参考坐标系.

    kwargs: 绘制的参数.

    返回: cartopy.mpl.feature_artist.FeatureArtist 实例用于绘制几何体.


+ **add_image**(factory, \*args, \*\*kwargs)

   向axes增加一个图像'工厂'

    一旦增加了图像'工厂', 它会被要求给定一个在绘制时的外包框,这样就不需要在加入图片时比较图片和地图的大小,只需要加入地图范围时比较.


+ **add_raster**(raster_source,\*\*slippy_image_kwargs)

    向axes添加栅格数据.


+ **add_wms**(wms, layers, wms_kwargs=None, \*\*kwargs)

    向axes增加一个WMS层(web map service). 需要owslib和PIL库.

    wms: string或者owslib.wms.WebMapService实例. web map的URL或者owslib WMS实例.4

    layers: string或者string的迭代器. 使用的layer名.

    wms_kwargs: dict或者None. 传给 WMSRasterSource 的getmap_extra_kwargs来定义getmap的时间参数.

    其他的参数传给matplotlib的imshow()方法.


+ **add_wmts**(wmts, layer_name, wmts_kwargs=None, \*\*kwargs)

    向地图增加一个地图切片服务层(Web Map Tile Service). 需要owslib和PIL库.

    wmts: 地图切片服务的URL, 或者一个owslib.wmts.WebMapTileService 实例.

    layer_name: 使用的layer名.

    wmts_kwargs: dict或None. 传给 WMTSRasterSource 的 gettile_extra_kwargs(如时间).

    其他的参数传给matplotlib的imshow()方法.


+ **background_img**(name='ne_shaded', resolution='low', extent=None, cache=False)

    从由CARTOPY_USER_BACKGROUNDS环境变量指定的目录中的预先存储好的图片添加到map中作为背景. 该目录可以通过func：self.read_user_background_images进行查看, 并且需要包含一个JSON文件来定义图像元数据.

    name: 需要读取的JSON文件中的图像名. 常用的有:‘ne_shaded’ : Natural Earth Shaded Relief;‘ne_grey’ : Natural Earth Grey Earth

    resolution:　需要读取的JSON文件中的图像的分辨率, 常见的有 ‘low’, ‘med’, ‘high’, ‘vhigh’, ‘full’.

    extent: 用一个高分辨率的背景, 然后缩放到一个小区域会需要很长时间去渲染图像因为图像会整个地被准备好, 即使只要用一小块. 指定extent可以只对特定的用[longitude start, longitude end,latitude start, latitude end]给出的区域进行渲染. 如[-11, 3, 48, 60] for the UK or [167.0, 193.0, 47.0, 68.0] to cross the date line.

    cache: 一个标志位来标志是否将加载的图像放在内存中. 图像将在被使用之前存在缓存中.


+ **background_patch** = None

    The patch that provides the filled background of the projection.


+ **outline_patch** = None

    The patch that provides the line bordering the projection.


+ **set_boundary**(path, transform=None, use_as_clip_path=True)

    Given a path, update the outline_patch and background_patch to take its shape.

    path: matplotlib.path.Path. The path of the desired boundary.

    transform: None or matplotlib.transforms.Transform. The coordinate system of the given path. Currently this must be convertible to data coordinates, and therefore cannot extend beyond the limits of the axes’ projection.

    use_as_clip_path: bool. Whether axes.patch should be updated. Updating axes.patch means that any artists subsequently created will inherit clipping from this path, rather than the standard unit square in axes coordinates.


+ **projection** = None

    The cartopy.crs.Projection of this GeoAxes.


+ **barbs**(x, y, u, v, \*args, \*\*kwargs)

    绘制一个barbs场.

    额外参数:

    transfrom: cartopy.crs.Prjection或者matplotlib transform. 矢量所在的源坐标系.

    regrid_shape: int或int的二元组. 如果给出的话箭头所在的点将会被内查到头一个看见的规则网格上. 如果是一个整数, 将会用来指定网格长度的最小值. 另一个维度将根据目标的宽高比进行放大. 如果给了一对整数, 将会分别决定网格的x,y方向的长度.

    target_exten: 4维tuple. 如果给定, 指定目标参考坐标系中有regrid_shape定义的规则网格的范围. 默认为当前地图投影的范围.


+ **quiver**(x, y, u, v, \*args, \*\*kwargs)

    绘制一个arrow场.

    参数同上.


+ **streamplot**(x, y, u, v, \*\*kwargs)

    绘制向量流向线.

    transfrom: cartopy.crs.Prjection或者matplotlib transform. 矢量所在的源坐标系.


+ **coastlines**(resolution='110m', color='black', \*\*kwargs)

    从Natual Earth的'coastline'shapefile文件中向axes家兔海岸轮廓线.

    resolution: 所用的Natural Earth数据集的分辨率,可以是'110m', '50m', '10m'


+ **gridlines**(crs=None, draw_labels=False, xlocs=None, ylocs=None, \*\*kwargs)

    在绘制地图时自动根据给定的坐标系画上坐标线.

    crs: 定义了gridlines的坐标系, 默认为cartopy.crs.PlateCarree.

    draw_labels: 像标记坐标轴一样在边缘标记坐标线.

    xlocs, ylocs: 一个坐标线位置迭代器或matplotlib.ticker.Locator实例, 用来确定坐标线在坐标系中的位置.

    其余控制line属性的关键字被传到matplotlib.collecions.Collection中.

    返回一个cartopy.mpl.gridliner.Gridliner实例.


+ **tissot**(rad_km=500000.0, lons=None, lats=None, n_samples=80, \*\*kwargs)

    向Axes加入Tissot’s indicatrices 蒂索指线(变形椭圆).

    rad_km: 所画圆的半径.

    lons: 一个numpy.ndarray, 记录了每个圆心的经度.

    lats: 一个numpy.ndarray, 记录了每个圆心的维度.

    n_samples: 每个圆周的整数个采样点数.


+ **get_extent**(crs=None)

    返回地图的范围(x0, x1, y0, y1)在给定坐标系中的值, 默认为当前Axes的坐标系.


+ **set_extent**(extents, crs=None)

    Set the extent (x0, x1, y0, y1) of the map in the given coordinate system.

    If no crs is given, the extents’ coordinate system will be assumed to be the Geodetic version of this axes’ projection.


+ **set_xticks**(ticks, minor=False, crs=None)

    Set the x ticks. 设置坐标轴刻度.

    ticks: 一个浮点数列表记录了需要标记的x轴刻度.

    minor: boolean flag indicating whether the ticks should be minor ticks i.e. small and unlabelled (default is False).

    crs: ticks值所在的坐标系,只支持从一个矩形坐标系转换到另一个矩形坐标系.


+ **set_yticks**(ticks, minor=False, crs=None)

    设置y轴刻度.


+ **stock_img**(name='ne_shaded')

    在地图上加上标准的图片, 目前只支持downsampled version of the Natural Earth shaded relief raster.


+ **set_global()**
    Set the extent of the Axes to the limits of the projection.


+ **hold_limits**(\*args, \*\*kwds)

    Keep track of the original view and data limits for the life of this context manager, optionally reverting any changes back to the original values after the manager exits.

    hold: bool, Whether to revert the data and view limits after the context manager exits.


+ **natural_earth_shp**(name='land', resolution='110m', category='physical', \*\*kwargs)

    将Natural Earth shapefile中的几何体作为pathCollection加到Axes中.


2. **cartopy.mpl.patch.path_to_geos**(path, force_ccw=False)

    Creates a list of Shapely geometric objects from a matplotlib.path.Path.

    path: matplotlib.path.Path实例.

    force_ccw: Boolean flag determining whether the path can be inverted to enforce ccw.

    返回: A list of shapely.geometry.polygon.Polygon,

        shapely.geometry.linestring.LineString and/or

        shapely.geometry.multilinestring.MultiLineString instances.

3. **cartopy.mpl.patch.geos_to_path**(shape)

    Creates a list of matplotlib.path.Path objects that describe a shape.

    shape: A list, tuple or single instance of any of the following types:

    shapely.geometry.point.Point,
    shapely.geometry.linestring.LineString,
    shapely.geometry.polygon.Polygon,
    shapely.geometry.multipoint.MultiPoint,
    shapely.geometry.multipolygon.MultiPolygon,
    shapely.geometry.multilinestring.MultiLineString,
    shapely.geometry.collection.GeometryCollection, or any type with a_as_mpl_path() method.

    Returns: A list of matplotlib.path.Path objects.


## 坐标网格线和刻度

Gridliner实例通常通过cartopy.mpl.geoaxes.GeoAxes.gridlines()方法由 cartopy.mpl.geoaxes.GeoAxes 实例创建., 它具有一系列的属性可以用来定制网格线和刻度.

**class cartopy.mpl.gridliner.Gridliner**(axes, crs, draw_labels=False, xlocator=None, ylocator=None, collection_kwargs=None)

    Object used by cartopy.mpl.geoaxes.GeoAxes.gridlines() to add gridlines and tick labels to a map.

输入参数:

    axes:  cartopy.mpl.geoaxes.GeoAxes 对象.

    crs: 定义了gridlines所在坐标系的cartopy.crs.CRS对象.

    draw_labels: 切换是否绘制标签. 为了更精细的控制, 网格线的属性可以单独修改.

    xlocator: matplotlib.ticker.Locator, 实例决定了坐标线在给定坐标系中的位置.

    ylocator: matplotlib.ticker.Locator, 实例决定了坐标线在给定坐标系中的位置.

    collection_kwargs: 字典格式的线条属性, 传给matplotlib.collections.Collection对象.

属性:

n_steps = None.
The number of interpolation points which are used to draw the gridlines.

xformatter = None.
The Formatter to use for the x labels.

xlabel_artists = None.
The x labels which were created at draw time.

xlabel_style = None.
A dictionary passed through to ax.text on x label creation for styling of the text labels.

xlabels_bottom = None.
Whether to draw labels on the bottom of the map.

xlabels_top = None.
Whether to draw labels on the top of the map.

xline_artists = None.
The x gridlines which were created at draw time.

xlines = None.
Whether to draw the x gridlines.

xlocator = None.
The Locator to use for the x gridlines and labels.

yformatter = None.
The Formatter to use for the y labels.

ylabel_artists = None.
The y labels which were created at draw time.

ylabel_style = None.
A dictionary passed through to ax.text on y label creation for styling of the text labels.

ylabels_left = None.
Whether to draw labels on the left hand side of the map.

ylabels_right = None.
Whether to draw labels on the right hand side of the map.

yline_artists = None.
The y gridlines which were created at draw time.

ylines = None.
Whether to draw the y gridlines.

ylocator = None.
The Locator to use for the y gridlines and labels.


```python
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import cartopy.crs as ccrs

from cartopy.mpl.gridliner import LONGITUDE_FORMATTER, LATITUDE_FORMATTER


ax = plt.axes(projection=ccrs.Mercator())
ax.coastlines()

gl = ax.gridlines(crs=ccrs.PlateCarree(), draw_labels=True,
                  linewidth=2, color='green', alpha=0.5, linestyle='--')
gl.xlabels_top = False
gl.ylabels_left = False
gl.xlines = False
gl.xlocator = mticker.FixedLocator([-180, -45, 0, 45, 180])
gl.xformatter = LONGITUDE_FORMATTER
gl.yformatter = LATITUDE_FORMATTER
gl.xlabel_style = {'size': 15, 'color': 'gray'}
gl.xlabel_style = {'color': 'red', 'weight': 'bold'}

plt.show()
```


![png](/images/Cartopy/output_92_0.png)


## Feature对象: 地图的各种要素

### Feature对象

**class cartopy.feature.Feature**(crs, \*\*kwargs)

代表了点,线和面的一个集合, 便于用来绘图和过滤操作.

crs: Feature所在的参考系.

方法:

geometries(): 返回该feature的一个几何体迭代器.

intersecting_geometries(extent): 返回和给定范围相交的几何体的迭代器.

### 从shapefile读取Feature

**class cartopy.feature.ShapelyFeature**(geometries, crs, \*\*kwargs)

feature的子类, 用于实现常见的功能, 如读取Natural Earth或GSHHS shapefile文件.

geometries: 几何形状的集合.

crs: 几何形状所在的参考系.

### Natural Earth数据接口

**class cartopy.feature.NaturalEarthFeature**(category, name, scale, \*\*kwargs)

A simple interface to Natural Earth shapefiles.

See http://www.naturalearthdata.com/

category: 数据集的类型, 如'cultural' or 'physical'.

name: 数据集的名称, 如'admin_0_boundary_lines_land'.

scale: 数据集的比例, '10m', '50m'或'110m'中的一个, 对应的比例尺分别为'1:10,000,000', '1:50,000,00' 和 '1:110,000,000'


```python
import cartopy.feature as cfeature
land_50m = cfeature.NaturalEarthFeature(category='physical', name='land', scale='50m',
                                        edgecolor='face',
                                        facecolor=cfeature.COLORS['land'])
```

### GSHHS数据接口

**class cartopy.feature.GSHHSFeature**(scale='auto', levels=None, \*\*kwargs)

An interface to the GSHHS dataset.

scale: 数据集的比例.  'auto','coarse','low','intermediate','high'或'full'.

levels: 一个1-4的整数列表代表绘制的GSHHS feature的级别,默认为[1]表示海岸线.

### 常用的Feature常量

为了简化使用, 预先定义了一些cartopy.feature常量. 这些预先定义好的Feature都是小比例尺(1:110m)的Natrual Earth数据集, 通过`GeoAxes.add_feature(cartopy.feature.BORDERS)`这样的方式添加.


|           Name            |        Description        |
| ------------------------- | ------------------------- |
| cartopy.feature.BORDERS   | 国家边界                  |
| cartopy.feature.COASTLINE | 海岸线,包括一些主要岛屿   |
| cartopy.feature.LAKES     | 自然和人工湖              |
| cartopy.feature.LAND      | 土地多边形,包括主要岛屿   |
| cartopy.feature.OCEAN     | 海洋多边形                |
| cartopy.feature.RIVERS    | 单线排水线,包括湖的中间线 |


常用的颜色也被存放在一个COLORS字典中:

```python
cartopy.feature.COLORS = {'water': array([ 0.59375 , 0.71484375, 0.8828125 ]), 'land': array([ 0.9375 , 0.9375 , 0.859375]), 'land_alt1': array([ 0.859375, 0.859375, 0.859375])}
```



```python
import cartopy.feature
sorted(cartopy.feature.COLORS.keys())
```




> ['land', 'land_alt1', 'water']




```python
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import cartopy.feature as cfeature
from matplotlib.offsetbox import AnchoredText


def main():
    fig = plt.figure(figsize=(10,10))
    ax = plt.axes(projection=ccrs.PlateCarree())
    ax.set_extent([80, 170, -45, 30])

    # Put a background image on for nice sea rendering.
    ax.stock_img()

    # Create a feature for States/Admin 1 regions at 1:50m from Natural Earth
    states_provinces = cfeature.NaturalEarthFeature(
        category='cultural',
        name='admin_1_states_provinces_lines',
        scale='50m',
        facecolor='none')

    SOURCE = 'Natural Earth'
    LICENSE = 'public domain'

    ax.add_feature(cfeature.LAND)
    ax.add_feature(cfeature.COASTLINE)
    ax.add_feature(states_provinces, edgecolor='green')

    # Add a text annotation for the license information to the
    # the bottom right corner.
    text = AnchoredText(r'$\mathcircled{{c}}$ {}; license: {}'
                        ''.format(SOURCE, LICENSE),
                        loc=4, prop={'size': 12}, frameon=True)
    ax.add_artist(text)

    plt.show()


if __name__ == '__main__':
    main()
```


![png](/images/Cartopy/output_97_0.png)


## Cartopy developer interfaces

## 设置cartopy
顶层的cartopy模块包含了config字典用来控制cartopy的行为.

**cartopy.config**

The config dictionary stores global configuration values for cartopy.

In the first instance, the config is defined in cartopy/__init__.py. It is possible to provide site wide customisations by including a siteconfig.py file along with the cartopy source code. siteconfig.py should contain a function called update_config which takes the config dictionary instance as its first and only argument (from where it is possible to update the dictionary howsoever desired).

For users without write permission to the cartopy source directory, a package called cartopy_userconfig should be made importable (consider putting it in site.getusersitepackages()) and should expose a function called update_config which takes the config dictionary as its first and only argument.

config字典中的参数:

pre_existing_data_dir: 存放标准数据(如Natural Earth数据)的绝对路径. 如果不存在则使用data_dir.

data_dir: 存放标准数据(如Natural Earth数据)的绝对路径. 如果数据不在其中并且是可以下载的, cartopy会从网上下载合适的数据到该文件夹的一个子文件夹中. 因此用户应该更改data_dir.

repo_data_dir: 随cartopy一起下载的数据的绝对路径, 通常只有OS包管理器和系统管理员进行设置.

downloaders: a dictionary mapping standard “specifications” to the appropriate Downloader. For further documentation and an example see cartopy.io.Downloader.from_config().

## 通过WMS和WMTS获取栅格数据


```python
"""
Interactive WMTS (Web Map Tile Service)
---------------------------------------

This example demonstrates the interactive pan and zoom capability
supported by an OGC web services Web Map Tile Service (WMTS) aware axes.

The example WMTS layer is a single composite of data sampled over nine days
in April 2012 and thirteen days in October 2012 showing the Earth at night.
It does not vary over time.

The imagery was collected by the Suomi National Polar-orbiting Partnership
(Suomi NPP) weather satellite operated by the United States National Oceanic
and Atmospheric Administration (NOAA).

"""

import cartopy.crs as ccrs
import matplotlib.pyplot as plt


def main():
    url = 'https://map1c.vis.earthdata.nasa.gov/wmts-geo/wmts.cgi'
    layer = 'VIIRS_CityLights_2012'

    ax = plt.axes(projection=ccrs.PlateCarree())
    ax.add_wmts(url, layer)
    ax.set_extent((-15, 25, 35, 60))

    plt.title('Suomi NPP Earth at night April/October 2012')
    plt.show()


if __name__ == '__main__':
    main()
```


![png](/images/Cartopy/output_101_0.png)


## Miscellaneous cartopy utilities
