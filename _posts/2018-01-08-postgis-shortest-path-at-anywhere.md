---
layout: post
author: Ding
title: 任意两点间的最短路径
date: 2018-01-08
categories: GIS
tags:
- PostGIS
- Shortest Path
---

* content
{:toc}

> 之前已经通过 Dijkstra 算法能够获得节点之间的最短路径,下面参照[博客](http://blog.csdn.net/yifei1989/article/details/14137891)完成地图上任意两点的最短路径的求解.







## 查询距起点和终点最近的两条线

起点: `'POINT(-122.205068 47.494881)'`

```sql
select edge_id, target, ST_DistanceSpheroid(the_geom, ST_GeomFromText('POINT(-122.205068 47.494881)', 4326), 'SPHEROID["WGS 84",6378137,298.257223563]')from ms.road
where  ST_DWithin(the_geom, ST_GeomFromText('POINT(-122.205068 47.494881)', 4326), 1000, 't')
order by ST_DistanceSpheroid(the_geom, ST_GeomFromText('POINT(-122.205068 47.494881)', 4326), 'SPHEROID["WGS 84",6378137,298.257223563]') limit 1
```

edge_id: `884148301986`, target:`13297`

终点: `'POINT(-122.308542 47.744709)'`

```sql
select edge_id, source, ST_DistanceSpheroid(the_geom, ST_GeomFromText('POINT(-122.308542 47.744709)', 4326), 'SPHEROID["WGS 84",6378137,298.257223563]')from ms.road
where  ST_DWithin(the_geom, ST_GeomFromText('POINT(-122.308542 47.744709)', 4326), 1000, 't')
order by ST_DistanceSpheroid(the_geom, ST_GeomFromText('POINT(-122.308542 47.744709)', 4326), 'SPHEROID["WGS 84",6378137,298.257223563]') limit 1
```

edge_id: `884147202987`, source:`108865`

## 在线上距离起点/终点最近的点

距起点最近的点: `'POINT(-122.20507083074 47.4956182056591)'`

```sql
select ST_AsText(
	ST_ClosestPoint(
	the_geom,
	ST_GeomFromText('POINT(-122.205068 47.494881)', 4326)))
from ms.road where edge_id = 884148301986;
```

距终点最近的点: `'POINT(-122.308453940561 47.745015689771)'`

```sql
select ST_AsText(
	ST_ClosestPoint(
	the_geom,
	ST_GeomFromText('POINT(-122.308542 47.744709)', 4326)))
from ms.road where edge_id = 884147202987;
```

### 求最短路径

起始节点为距离起点最近的线的target.

目标节点为距离终点最近的线的source.

```sql
SELECT seq, id1 AS node, id2 AS edge, cost
  FROM pgr_dijkstra('
  SELECT gid as id,  
  source::integer,  
  target::integer,  
  length::double precision as cost  
  FROM ms.road',  
  13297, 108865, false, false)
```

### 拼接几部分成为最终的路径

第一部分: 起点---最近路上的点---起点最近的路的target

第二部分: 起点最近的路的target---终点最近的路的source

第三部分: 终点最近的路的source---最近路上的点---终点


+ 第二部分

```sql
SELECT ST_AsText(ST_LineMerge(ST_Union(the_geom)))
	FROM
	(SELECT seq, id1 AS node, id2 AS edge, cost
  	FROM pgr_dijkstra('
  	SELECT gid as id,  
  	source::integer,  
  	target::integer,  
  	length::double precision as cost  
  	FROM ms.road',  
  	13297, 108865, false, false)) p1,
	ms.road p2
	WHERE p1.edge=p2.gid
;
```

结果: `'LINESTRING(-122.202209830284 47.4956291913986,-122.207099497318 47.4956104159355,-122.207099497318 47.4938508868217,-122.20744818449 47.4927297234535,-122.207440137863 47.4920806288719,-122.207440137863 47.4918392300606,-122.207429409027 47.4912303686142,- (...)'`

+ 拼接第一,二,三部分:

```sql
-- 暂时不太对
SELECT  ST_LineMerge(ST_Union(array[
	SELECT ST_LineMerge(ST_Union(the_geom)) as geom
		FROM
		(SELECT seq, id1 AS node, id2 AS edge, cost
	  	FROM pgr_dijkstra('
	  	SELECT gid as id,  
	  	source::integer,  
	  	target::integer,  
	  	length::double precision as cost  
	  	FROM ms.road',  
	  	101826, 67796, false, false)) p1,
		ms.road p2
		WHERE p1.edge=p2.gid,
  select ST_GeomFromEWKB(the_geom) from ms.road where edge_id = 884148301986

  --select ST_GeomFromEWKB(the_geom) from ms.road where edge_id = 884147202987
]))
```

+ 找出起点终点在线上的百分比并截取出相应路段:

```sql
select ST_Line_Locate_Point(
ST_LineMerge(ST_Union(the_geom)),
ST_GeomFromText('POINT(-122.20507083074 47.4956182056591)',4326))
from (SELECT seq, id1 AS node, id2 AS edge, cost
  	FROM pgr_dijkstra('
  	SELECT gid as id,  
  	source::integer,  
  	target::integer,  
  	length::double precision as cost  
  	FROM ms.road',  
  	13297, 108865, false, false)) p1,
	ms.road p2
	WHERE p1.edge=p2.gid;
```

起点所在百分比位置:`0.00839316152450122`

```sql
select ST_Line_Locate_Point(
ST_LineMerge(ST_Union(the_geom)),
ST_GeomFromText('POINT(-122.308453940561 47.745015689771)',4326))
from (SELECT seq, id1 AS node, id2 AS edge, cost
  	FROM pgr_dijkstra('
  	SELECT gid as id,  
  	source::integer,  
  	target::integer,  
  	length::double precision as cost  
  	FROM ms.road',  
  	13297, 108865, false, false)) p1,
	ms.road p2
	WHERE p1.edge=p2.gid;
```

终点所在百分比位置:`0.995896693587171`

```sql
INSERT INTO ms.path (the_geom)
SELECT ST_LineSubstring(
ST_LineMerge(ST_Union(the_geom)),
0.00839316152450122,
0.995896693587171)
from (SELECT seq, id1 AS node, id2 AS edge, cost
  	FROM pgr_dijkstra('
  	SELECT gid as id,  
  	source::integer,  
  	target::integer,  
  	length::double precision as cost  
  	FROM ms.road',  
  	13297, 108865, false, false)) p1,
	ms.road p2
	WHERE p1.edge=p2.gid;
```

![任意点最短路径](/images/postgis-shortest-path/任意点最短路径.png)

放大看起点和终点的细节:

![任意点最短路径_起点](/images/postgis-shortest-path/任意点最短路径_起点.png)

![任意点最短路径_终点](/images/postgis-shortest-path/任意点最短路径_终点.png)

## 任意两点最短路径函数

```sql
create function _myShortPath(startx float, starty float,endx float,endy float)   
returns geometry as  
$body$
declare  
    v_startLine geometry;--离起点最近的线  
    v_endLine geometry;--离终点最近的线  

    v_startTarget integer;--距离起点最近线的终点  
    v_endSource integer;--距离终点最近线的起点  

    v_statpoint geometry;--在v_startLine上距离起点最近的点  
    v_endpoint geometry;--在v_endLine上距离终点最近的点  

    v_res geometry;--最短路径分析结果  


    v_perStart float;--v_statpoint在v_res上的百分比  
    v_perEnd float;--v_endpoint在v_res上的百分比  

    v_shPath geometry;--最终结果  
begin     

    --查询离起点最近的线  
    select the_geom, target into v_startLine, v_startTarget from ms.road where
    ST_DWithin(the_geom, ST_GeomFromText('POINT('|| startx ||' ' || starty ||')', 4326), 1000, 't')
    order by ST_DistanceSpheroid(the_geom, ST_GeomFromText('POINT('|| startx ||' ' || starty ||')', 4326),
	'SPHEROID["WGS 84",6378137,298.257223563]') limit 1;

    --查询离终点最近的线  
    select the_geom, source into v_endLine, v_endSource from ms.road where
    ST_DWithin(the_geom, ST_GeomFromText('POINT('|| endx ||' ' || endy ||')', 4326), 1000, 't')
    order by ST_DistanceSpheroid(the_geom, ST_GeomFromText('POINT('|| endx ||' ' || endy ||')', 4326),
	'SPHEROID["WGS 84",6378137,298.257223563]') limit 1;

    --如果没找到最近的线，就返回null  
    if (v_startLine is null) or (v_endLine is null) then  
        return null;  
    end if ;  

    select  ST_ClosestPoint(v_startLine, ST_GeomFromText('POINT('|| startx ||' ' || starty ||')', 4326)) into v_statpoint;  
    select  ST_ClosestPoint(v_endLine, ST_GeomFromText('POINT('|| endx ||' ' || endy ||')', 4326)) into v_endpoint;  


    --最短路径  
    --SELECT st_linemerge(st_union(b.geom)) into v_res  
    --FROM pgr_kdijkstraPath(  
    --'SELECT id, source, target, ' || costfile ||' as cost FROM route',  
    --v_startTarget, array[v_endSource], true, false  
    --) a,  
    --route b  
    --WHERE a.id3=b.id  
    --GROUP by id1  
    --ORDER by id1;

    SELECT ST_LineMerge(ST_Union(p2.the_geom))
    FROM pgr_dijkstra(
    'SELECT gid as id, source::integer, target::integer, length::double precision as cost FROM ms.road',
    v_startTarget, v_endSource, false, false) p1,
    ms.road p2
    WHERE p1.id2=p2.gid;


    --如果找不到最短路径，就返回null  
    if(v_res is null) then  
        return null;  
    end if;  

    --将v_res,v_startLine,v_endLine进行拼接  
    select  st_linemerge(ST_Union(array[v_res,v_startLine,v_endLine])) into v_res;  

    select  ST_Line_Locate_Point(v_res, v_statpoint) into v_perStart;  
    select  ST_Line_Locate_Point(v_res, v_endpoint) into v_perEnd;  

    --截取v_res  
    SELECT ST_Line_SubString(v_res,v_perStart, v_perEnd) into v_shPath;  

    return v_shPath;  



end;  
$body$  
LANGUAGE plpgsql VOLATILE STRICT  
```
