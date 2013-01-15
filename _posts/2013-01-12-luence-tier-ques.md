---
layout: post
title: lucene Cartesian tier boxes地理位置检索问题
category: default
---
1. lucene spatial3.2 版本支持geohash 和 Cartesian tier boxes位置索引

    geohash 见 [http://en.wikipedia.org/wiki/Geohash](http://en.wikipedia.org/wiki/Geohash)   
    Cartesian tier boxes见 [http://www.nsshutdown.com/projects/lucene/whitepaper/locallucene_v2.html](http://www.nsshutdown.com/projects/lucene/whitepaper/locallucene_v2.html)  
    
    目前我们索引采用 Cartesian tier boxes 


2.  传入DistanceQueryBuilder 的距离是整个检索 box的宽度和高度，不是圆形的半径

3.   测试宽度等于10公里的检索，如下图  
    <img src="http://img.haowaimai.mobi/yqg/luence1.png" />
    中间是检索的坐标点A，边上是应该被检索出的点B。10公里内覆盖取得的boxid 为 329.00385, 329.00386， B点所在的boxid 为330.00386
    上述A点检索一直无法出现B点，直到扩大到35公里范围，boxid才覆盖到如下：   
    329.00384  
    329.00385  
    329.00386  
    330.00384  
    330.00385  
    330.00386  
    这时B点才会被检索出来，这个应该是box投影算法在大于10公里出现问题

    测试代码如下:  
    
    <pre>
        IProjector projector = new SinusoidalProjector();
        Analyzer analyzer = new StandardAnalyzer(Version.LUCENE_32);
        CartesianTierPlotter ctp = new CartesianTierPlotter(0, projector, CartesianTierPlotter.DEFALT_FIELD_PREFIX);
        int startTier = ctp.bestFit(15 / RATE_MILE_TO_KM);
        int endTier = ctp.bestFit(3 / RATE_MILE_TO_KM);
        // 测试边框索引 ，边框具体算法见 http://www.nsshutdown.com/projects/lucene/whitepaper/locallucene_v2.html
        for (int i = startTier; i <= endTier; ++i) {
            CartesianTierPlotter ctpt = new CartesianTierPlotter(i, projector, CartesianTierPlotter.DEFALT_FIELD_PREFIX);
            System.out.println("index boxid " + i + " tier : " + ctpt.getTierBoxId(31.239853, 121.49736));
        }
        CartesianPolyFilterBuilder polyFilterBuilder = new DistanceTest.CartesianPolyFilterBuilder(
                                                                                                   CartesianTierPlotter.DEFALT_FIELD_PREFIX,
                                                                                                   startTier, endTier);
        Shape shape = polyFilterBuilder.getBoxShape(31.2214, 121.428, km2Mile(35));
        System.out.println("search index boxid : ");
        for (Double area : shape.getArea()) {
            System.out.println(area);
        }
    </pre>


4.  开始调试看能否扩大检索范围，发现 CartesianPolyFilterBuilder 中getShapeLoop似乎是个bug,如下图
     <img src="http://img.haowaimai.mobi/yqg/luence2.png" />

     即使在上面打上patch 大于10公里的检索仍然不是很准确

5. 从3.4 一直升级到3.61 ，上面问题仍然没有解决。 最后升级到4.0，因为API变动比较大，改动比较多，最终修复该问题

{% include references.md %}