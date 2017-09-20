> 四色定理（世界近代三大数学难题之一），又称四色猜想、四色问题，是世界三大数学猜想之一。世界三大数学猜想即费马猜想、四色猜想和哥德巴赫猜想。
>
> 费马猜想于1994年由英国数学家安德鲁·怀尔斯完成,遂称费马大定理。
>
> 四色猜想的证明于1976年由美国数学家阿佩尔与哈肯借助计算机完成，遂称四色定理；
>
> 哥德巴赫猜想尚未解决，目前最好的成果（陈氏定理）乃于1966年由中国数学家陈景润取得。
>
> 哥德巴赫猜想就是传说中的数学民科聚集地，同样，四色定理由于尚未有纯理论证明，仍有很多民科正孜孜不倦的进行着尝试。



在学习cartopy是尝试通过shapefile文件绘制世界地图， 想到了四色问题，于是尝试着完成一次程序验证。

首先想到的是怎样判断两国之间有共同边界，幸好shapely中的基础类base定义了touch方法可以判断是否有公共边。

考虑到直接使用touch方法进行两两比较在一共有200多个国家和地区是要进行上万次运算，效率还是比较低的，所以尝试建立rtree索引。

美国出现问题

换策略，邻接矩阵

## 四色定理的一个验证程序