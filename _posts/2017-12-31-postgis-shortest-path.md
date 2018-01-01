---
layout: post
author: Ding
title: 基于postgis的最短路径查询
date: 2017-12-31
categories: PostGis
tags:
- PostGis
- Shortest Path
---

* content
{:toc}

> 用postgis做最短路径查询






## 第一个例子

### 建立test数据库并安装相关空间插件

```sql
CREATE DATABASE test
  WITH OWNER = postgres
       ENCODING = 'UTF8'
       TABLESPACE = pg_default
       LC_COLLATE = 'zh_CN.UTF-8'
       LC_CTYPE = 'zh_CN.UTF-8'
       CONNECTION LIMIT = -1;

CREATE EXTENSION postgis;
CREATE EXTENSION postgis_sfcgal;
CREATE EXTENSION pgrouting;
CREATE EXTENSION plpgsql;
```


### 创建数据表并插入数据

```sql
CREATE TABLE edge_table ( id serial,
dir character varying,
source integer,
target integer,
cost double precision, reverse_cost double precision, x1 double precision,
y1 double precision, x2 double precision, y2 double precision, the_geom geometry
);

INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 2,0, 2,1);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES (-1, 1, 2,1, 3,1);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES (-1, 1, 3,1, 4,1);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 2,1, 2,2);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1,-1, 3,1, 3,2);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 0,2, 1,2);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 1,2, 2,2);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 2,2, 3,2);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 3,2, 4,2);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 2,2, 2,3);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1,-1, 3,2, 3,3);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1,-1, 2,3, 3,3);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1,-1, 3,3, 4,3);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 2,3, 2,4);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 4,2, 4,3);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 4,1, 4,2);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 0.5,3.5, 1.999999999999,3);
INSERT INTO edge_table (cost,reverse_cost,x1,y1,x2,y2) VALUES ( 1, 1, 3.5,2.3, 3.5,4);
```

这里把(x1, y1)和(x2, y2)作为线的起点和终点，生成线数据图层。 根据cost和reverse_cost 决定是双向路还是单行道，
分别用'B','FT'和'TF'标记。

```sql
UPDATE edge_table SET the_geom = st_makeline(st_point(x1,y1),st_point(x2,y2)),
dir = CASE WHEN (cost>0 and reverse_cost>0) THEN 'B' -- both ways
WHEN (cost>0 and reverse_cost<0) THEN 'FT' -- direction of the L
WHEN (cost<0 and reverse_cost>0) THEN 'TF' -- reverse direction
ELSE '' END;
```

结果为：

![数据](/images/postgis-shortest-path/data_table.png)

在 QGIS 中连接 postgis 数据库，并显示标签为 id

![qgis展示](/images/postgis-shortest-path/data_in_qgis.png)

### 创建格式化拓扑数据

使用`pgrouting`的函数生成`source`和`target`列。

```sql
SELECT pgr_createTopology('edge_table',0.001);
```

SQL结果为：

![pgr_createTopology](/images/postgis-shortest-path/pgr_createTopology.png)

执行后生成一个表` edge_table_vertices_pgr`. 其中包含了道路数据的交点，也在 edge_table 表的 `source` 和 `target` 列中表现. 他们是一组点的组合.

### 计算最短路径

我们用 pgr_dijkstra 函数计算出, edge_table_vertices_pgr 中点 1 到 11 间的最短距离.

改函数的实现核心是 Dijkstra 算法，它又叫迪科斯彻算法（Dijkstra），算法解决的是有向图中单个源点到其他顶点的最短路径问题。举例来说，如果图中的顶点表示城市，而边上的权重表示著城市间开车行经的距离.
Dijkstra 算法可以用来找到两个城市之间的最短路径。

计算节点 1 到 11 的最短距离：

```sql
SELECT seq, id1 AS node, id2 AS edge, cost FROM
pgr_dijkstra('
  SELECT  id AS id,
  source::integer,
  target::integer,
  cost::double precision AS cost
  FROM edge_table',
1, 11, false, false);
```

结果为：

![dijkstra_result](/images/postgis-shortest-path/dijkstra_result.png)

意思为节点 1-11 的最短路径为 `1-2-5-10-11`，经过的路径为 `1-4-10-12`

### 在 QGis 中展示查询结果

由于 QGIS 不支持动态查询数据库，只能够从数据库中读取数据，所以需要将查询结果生成一张数据表存在数据库中，再通过 QGIS 进行展示。


```sql
create table line(id serial,the_geom geometry);
insert into line (the_geom) select ST_MakeLine(ARRAY (select the_geom
from (SELECT seq, id1 AS node FROM pgr_dijkstra(
'SELECT  id AS id,
source::integer,
target::integer,
cost::double precision AS cost
FROM edge_table', 1, 11, false, false))path, edge_table_vertices_pgr p
where path.node = p.id order by seq));
```

在 QGIS 中加载数据，如图所示：

![dijkstra_result_inQGIS](/images/postgis-shortest-path/dijkstra_result_inQGIS.png)

## 测试成都数据

### 通过 sha2pgsql 导入成都道路数据

![导入成都数据](/images/postgis-shortest-path/导入成都数据.png)

### 生成道路网

+ 生成`source`，`target`和`length`字段

```sql
ALTER TABLE chengdu ADD COLUMN source integer;  
ALTER TABLE chengdu ADD COLUMN target integer;  
ALTER TABLE chengdu ADD COLUMN length double precision;
-- 经纬度距离
UPDATE chengdu SET length = ST_Length(geom);
-- 曲面距离
UPDATE chengdu SET length = ST_LengthSpheroid(geom, 'SPHEROID["WGS 84",6378137,298.257223563]');
```

+ 生成道路网的拓扑节点

确认空间扩展插件已安装：

```sql
CREATE EXTENSION postgis;
CREATE EXTENSION postgis_topology;
CREATE EXTENSION fuzzystrmatch;
```

生成道路之间的节点， `0.001`和`0.1`为道路节点的容差，不要太大也不要太小：

```sql
select pgr_createTopology('chengdu',0.001,source:='source',id:='gid',target:='target',the_geom:='geom');
select pgr_createTopology('chengdu',0.01,source:='source',id:='gid',target:='target',the_geom:='geom');
```

![生成拓扑结构](/images/postgis-shortest-path/生成拓扑结构.png)

结果生成了一张成都道路网的节点数据表 `chengdu_vertices_pgr`，在QGIS中查看结果：

设定容差为 0.001:
![成都市道路节点](/images/postgis-shortest-path/成都市道路节点.png)

设定容差为 0.01:
![成都市道路节点0.01](/images/postgis-shortest-path/成都市道路节点_0.01.png)

设定容差为 0.0005:
![成都市道路节点0.0005](/images/postgis-shortest-path/成都市道路节点_0.0005.png)

### 最短路径查询

在 QGIS 中把 `chengdu_vertices_pgr` 图层显示图层标签为`id`，查询两个节点`1152`和`9932`的最短路径：

```sql
SELECT seq, id1 AS node, id2 AS edge, cost
FROM pgr_dijkstra('
	SELECT gid as id,  
	source::integer,  
	target::integer,  
	length::double precision as cost  
	FROM chengdu',  
	1152, 9932, false, false);
```

![成都市最短路查询](/images/postgis-shortest-path/成都市最短路查询.png)

### 在 QGIS 中显示查询结果

```sql
create table chengdu_path_result(id serial,the_geom geometry);
insert into chengdu_path_result (the_geom)
select ST_MakeLine(ARRAY (select the_geom
	from (SELECT seq, id1 AS node, id2 AS edge, cost
		FROM pgr_dijkstra('
		SELECT gid as id,  
		source::integer,  
		target::integer,  
		length::double precision as cost  
		FROM chengdu',  
		1152, 9932, false, false)) path, chengdu_vertices_pgr p
	where path.node = p.id order by seq));
```

在 QGIS 中打开：

![成都最短路结果](/images/postgis-shortest-path/成都最短路结果.png)


## 参考

[PostgreSQL 路径规划插件 pgruoting 介绍](https://yq.aliyun.com/articles/256).
