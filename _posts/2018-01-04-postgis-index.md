---
layout: post
author: Ding
title: 在 PostGIS 中使用索引来加速空间查询
date: 2018-01-04
categories: GIS
tags:
- Index
- PostGIS
---

* content
{:toc}

> 对空间数据 建立索引






## 建立 GiST 索引

参考[博客](https://yq.aliyun.com/articles/175035) 选个 GiST 索引。


```sql
CREATE EXTENSION btree_gist;
CREATE INDEX gist ON ms.road USING GIST (the_geom);
```



## 查询

GPS 点一定距离范围内的 road 按找距离远近进行排序。

```sql
select edge_id, ST_DistanceSpheroid(the_geom, ST_GeomFromText('POINT(-122.1070833 47.66748333)', 4326), 'SPHEROID["WGS 84",6378137,298.257223563]')from ms.road
where  ST_DWithin(the_geom, ST_GeomFromText('POINT(-122.1070833 47.66748333)', 4326), 100, 't')
order by ST_DistanceSpheroid(the_geom, ST_GeomFromText('POINT(-122.1070833 47.66748333)', 4326), 'SPHEROID["WGS 84",6378137,298.257223563]')
```


使用到的 postgis 函数定义：

```sql
-- Returns minimum distance in meters between two lon/lat geometries given a particular spheroid.
float ST_DistanceSpheroid(geometry geomlonlatA, geometry geomlonlatB, spheroid measurement_spheroid);

-- Returns true if the geometries are within the specified distance of one another.
boolean ST_DWithin(geometry g1, geometry g2, double precision distance_of_srid);
boolean ST_DWithin(geography gg1, geography gg2, double precision distance_meters);
boolean ST_DWithin(geography gg1, geography gg2, double precision distance_meters, boolean use_spheroid);

-- Constructs a PostGIS ST_Geometry object from the OGC Well-Known text representation.
geometry ST_GeomFromText(text WKT);
geometry ST_GeomFromText(text WKT, integer srid);
```

## 查询 Candidate Roads

```python
import psycopg2
import pandas as pd
import pickle

file = 'gps_data.csv'
#data = pd.read_csv(file , sep = '	', dtype ={'Latitude':str,'Longtitude':str})

conn = psycopg2.connect(database="test", user="postgres",
                        password="********", host="127.0.0.1",
                        port="5432")
cur = conn.cursor()
KnnSql = '''select edge_id, ST_DistanceSpheroid(the_geom, ST_GeomFromText(%s, 4326), 'SPHEROID["WGS 84",6378137,298.257223563]')from ms.road
where  ST_DWithin(the_geom, ST_GeomFromText(%s, 4326), 100, 't')
order by ST_DistanceSpheroid(the_geom, ST_GeomFromText(%s , 4326), 'SPHEROID["WGS 84",6378137,298.257223563]')'''

f = open(file, 'r')
f.readline()

candidate_roads = []
while f:
    line = f.readline().split('\t')
    point = 'Point(' + line[3] + ' ' + line[2] + ')'    
    cur.execute(KnnSql, (point, point, point))
    result = cur.fetchall()
    candidate_roads.append(result)
```

## Candidate Roads 上的最近点


Line 上与 Point 最近的点：

```sql
select  ST_AsText(
ST_ClosestPoint(
ST_GeomFromText('LINESTRING(-122.109748721123 47.6673012971878, -122.109268605709 47.6675400137901, -122.109078168869 47.6675695180893, -122.107079923153 47.6675292849541, -122.105398178101 47.6675292849541)', 4326),
ST_GeomFromText('POINT(-122.1070833 47.66748333)', 4326)
))

"POINT(-122.107082373739 47.6675293342948)"
```


```sql
select ST_DistanceSpheroid(
ST_GeomFromText('POINT(-122.1070833 47.66748333)', 4326),
ST_GeomFromText('POINT(-122.72741317749 47.900390625)', 4326),
'SPHEROID["WGS 84",6378137,298.257223563]'
)

5.11540771568587
```

对比直接计算点和线的距离:

```sql
select ST_DistanceSpheroid(
ST_GeomFromText('POINT(-122.1070833 47.66748333)', 4326),
ST_GeomFromText('LINESTRING(-122.109748721123 47.6673012971878, -122.109268605709 47.6675400137901, -122.109078168869 47.6675695180893, -122.107079923153 47.6675292849541, -122.105398178101 47.6675292849541)', 4326),
'SPHEROID["WGS 84",6378137,298.257223563]')

5.11473800255717
```
