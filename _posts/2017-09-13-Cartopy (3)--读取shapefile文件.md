---
layout: post
author: Ding
title:  "Cartopy Document--读取shapefile文件"
date:   2017-09-13
categories: python cartopy
tags: python cartopy matplotlib visualising
---

* content
{:toc}

> Starting in 2016, Basemap came under new management. The Cartopy project will replace Basemap, but it hasn’t yet implemented all of Basemap’s features. All new software development should try to use Cartopy whenever > possible, and existing software should start the process of switching over to use Cartopy. All maintenance and development efforts should be focused on Cartopy.
>
> 这是cartopy 的简略中文文档








## Using the cartopy shapereader

### shapereader模块
Cartopy基于pyshp模块提供了一个面向对象的shapefile文件阅读器, 可以简易的读取标准矢量数据集.

**class cartopy.io.shapereader.Reader**(filename)

    一个Reader实例最基本的方法是records()和geometries().

    geometries()
        返回一个shapefile文件中的几何体的迭代器. 如果相关的元数据已知,可以通过这个方法获得shapefile中的几何体,佛贼用records()方法.

    records()
        返回一个Record实例的迭代器.

**class cartopy.io.shapereader.Record**(shape, geometry_factory, attributes, fields)

    shapefile中的一个逻辑实体, 包括几何体和相关的属性.

    attributes = None, 一个属性名称到属性值的字典.

    bounds, Record几何体的边界.

    geometry, 该Record的几何体实例. 如果shapefiel中定义了一个null shape那么geometry为None.

### Helper functions for shapefile acquisition
对于经常使用的数据如GSHHS数据集和NaturalEarthData网站, cartopy提供了连接的接口. 这些接口可以自动的获取数据,如果在硬盘上没有, 它会从合适的源获取(通常是在英特网上下载).

**cartopy.io.shapereader.natural_earth**(resolution='110m', category='physical', name='coastline')

    返回需求的natural earth shapefile的路径,如果需要的话将会下载和解压.

**cartopy.io.shapereader.gshhs**(scale='c', level=1)

    返回需求的GSHHS shapefile的路径,如果需要的话将会下载和解压.

### Using the shapereader
我们可以通过natural_earth函数从http://www.naturalearthdata.com/downloads/110m-cultural-vectors/110m-admin-0-countries/ 获取Natural Earth found提供的国家数据集.


```python
import cartopy.io.shapereader as shpreader

shpfilename = shpreader.natural_earth(resolution='110m',
                                      category='cultural',
                                      name='admin_0_countries')
# /home/ding/.local/share/cartopy/shapefiles/natural_earth/cultural/110m_admin_0_countries.shp
```

然后可以通过Reader读取shapefile中的第一个国家.


```python
reader = shpreader.Reader(shpfilename)
countries = reader.records()
country = next(countries)
```

可以通过Record.attributes获取国家的属性字典.


```python
print (type(country.attributes))
print (sorted(country.attributes.keys()))
```

    <class 'dict'>
    ['abbrev', 'abbrev_len', 'adm0_a3', 'adm0_a3_is', 'adm0_a3_un', 'adm0_a3_us', 'adm0_a3_wb', 'adm0_dif', 'admin', 'brk_a3', 'brk_diff', 'brk_group', 'brk_name', 'continent', 'economy', 'featurecla', 'fips_10', 'formal_en', 'formal_fr', 'gdp_md_est', 'gdp_year', 'geou_dif', 'geounit', 'gu_a3', 'homepart', 'income_grp', 'iso_a2', 'iso_a3', 'iso_n3', 'labelrank', 'lastcensus', 'level', 'long_len', 'mapcolor13', 'mapcolor7', 'mapcolor8', 'mapcolor9', 'name', 'name_alt', 'name_len', 'name_long', 'name_sort', 'note_adm0', 'note_brk', 'pop_est', 'pop_year', 'postal', 'region_un', 'region_wb', 'scalerank', 'sov_a3', 'sovereignt', 'su_a3', 'su_dif', 'subregion', 'subunit', 'tiny', 'type', 'un_a3', 'wb_a2', 'wb_a3', 'wikipedia', 'woe_id']


我们可以通过下面的代码找到人口最少的五个国家.


```python
reader = shpreader.Reader(shpfilename)

# define a function which returns the population given the country
population = lambda country: country.attributes['pop_est']

# sort the countries by population and get the first 5
countries_by_pop = sorted(reader.records(), key=population)[:5]

', '.join([country.attributes['name_long'] for country in countries_by_pop])
```




    'Western Sahara, French Southern and Antarctic Lands, Falkland Islands, Antarctica, Greenland'
