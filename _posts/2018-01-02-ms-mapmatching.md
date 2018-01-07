---
layout: post
author: Ding
title: ms-mapmatching论文提供的数据集
date: 2018-01-02
categories: GIS
tags:
- map matching
- PostGIS
---

* content
{:toc}

> 论文地址以及相关数据在[这里](https://www.microsoft.com/en-us/research/publication/hidden-markov-map-matching-noise-sparseness/)





## 准备数据

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

![检查数据](/images/map-matching/检查数据.png)


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

![道路数据](/images/map-matching/道路数据.png)

### 生成道路节点

```sql
ALTER TABLE ms.road ADD COLUMN length double precision;
UPDATE ms.road SET length = ST_LengthSpheroid(the_geom, 'SPHEROID["WGS 84",6378137,298.257223563]');
select pgr_createTopology('ms.road',0.0001,source:='source_id',id:='edge_id',target:='target_id',the_geom:='the_geom',clean:='true');
```

![道路节点](/images/map-matching/道路节点.png)

### 轨迹数据

+ 原始GPS轨迹数据

先用 QGIS 读取 `gps_data.csv` 文件，然后导出为 shp 文件再通过 shp2psql 工具导入 postgis 数据库。

+ 真实轨迹数据

```python
import psycopg2
import pandas as pd
conn = psycopg2.connect(database="test", user="postgres",
                        password="********", host="127.0.0.1",
                        port="5432")

cur = conn.cursor()

data = pd.read_csv('ground_truth_path.csv',
                   sep = '	')

SQL = '''
CREATE TABLE ms.ground_truth(edge_id bigint, traversed boolean);
SELECT AddGeometryColumn('ms','ground_truth', 'the_geom',4326,'LINESTRING',2);
'''
cur.execute(SQL)
conn.commit()

insert_query = '''INSERT INTO ms.ground_truth
VALUES (%s, %s, %s);
'''

for i in range(len(data)):
    edge_id = str(data.loc[i][0])
    traversed = str(data.loc[i][1])
    cur.execute('SELECT the_geom FROM ms.road WHERE edge_id = %s;', (edge_id,))
    geom = cur.fetchone()[0]
    cur.execute(insert_query, (edge_id, traversed, geom))
    print('line:' + str(i))

conn.commit()
```
真实路径如图所示：

![真实路径](/images/map-matching/ground_truth.png)

放大观察细节可以发现 GPS 轨迹点和真实路径的差别：

![路径细节](/images/map-matching/ground_truth_detail.png)
