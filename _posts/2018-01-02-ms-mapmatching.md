---
layout: post
author: Ding
title: ms-mapmatching论文提供的数据集
date: 2017-01-02
categories: GIS
tags:
- map matching
- PostGIS
---

* content
{:toc}

> 论文地址以及相关数据在[这里](https://www.microsoft.com/en-us/research/publication/hidden-markov-map-matching-noise-sparseness/)





##　准备数据

### 道路数据表

```sql
DROP TABLE ms.road;
DROP SCHEMA ms;
-- 创建 schema
CREATE SCHEMA ms;
-- 创建 table
CREATE TABLE ms.road(edge_id bigint, source_id bigint, target_id bigint, two_way boolean, speed double precision, vertex_count integer);
SELECT AddGeometryColumn('ms','road', 'the_geom',4326,'LINESTRING',2);

INSERT INTO ms.road(edge_id, source_id, target_id, two_way, speed, vertex_count, the_geom)
VALUES (883991900000,883991900000,883991900001,'1',22.22222222,10, ST_GeomFromText('LINESTRING(-122.732318937778 47.8899192810059, -122.732139229774 47.8903403878212, -122.731820046902 47.8910404443741, -122.731310427189 47.8921294212341, -122.730749845505 47.8933900594711, -122.730208039284 47.894441485405, -122.729588449001 47.8957316279411, -122.729159295559 47.8968098759651, -122.727560698986 47.9000902175903, -122.72741317749 47.900390625)', 4326)
);
```

在 QGIS 中看一下是否正确插入了数据：

![检查数据](/home/ding/Documents/git/Dingxxxx.github.io/images/map-matching/检查数据.png)


编写 python 脚本批量插入道路数据到 postgis 数据库中：

```python
import psycopg2
import pandas as pd
conn = psycopg2.connect(database="test", user="postgres",
                        password="********", host="127.0.0.1",
                        port="5432")

cur = conn.cursor()

data = pd.read_csv('road_network.txt',
                   sep = '	')

insert_query = '''INSERT INTO ms.road
VALUES (%s,%s,%s,%s,%s,%s,ST_GeomFromText(%s, 4326));
'''

for i in range(len(data)):
    line = data.loc[i]

    edge_id = str(data.loc[i]['edge_id'])
    source_id = str(data.loc[i]['source_id'])
    target_id = str(data.loc[i]['target_id'])
    two_way = str(data.loc[i]['two_way'])
    speed = str(data.loc[i]['speed'])
    vertex_count = str(data.loc[i]['vertex_count'])
    the_geom = str(data.loc[i]['the_geom'])

    cur.execute(insert_query, (edge_id,source_id,target_id,two_way,speed,
                               vertex_count,the_geom))

    print('line:' + str(i))
# 一共158166行道路数据
```

![道路数据](/home/ding/Documents/git/Dingxxxx.github.io/images/map-matching/道路数据.png)


### 轨迹数据

+ 原始GPS轨迹数据

+ 真实轨迹数据
