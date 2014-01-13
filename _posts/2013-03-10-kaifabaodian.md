---
layout: post
title: 日常开发宝典
category: dev
---

*   如果线上出问题，需要重启应用，可用kill -3 pid 打出堆栈日志，保留现场以便后续日志分析
*   Velocity使用时注意循环对同个变量赋值问题，如果中间某次循环为空，该值保留上次循环结果
*   注意本地编码问题，win下默认GBK，ubuntu下默认utf-8，对于url中文传参、byte string互转、cache key需要注意
*   wget http请求带参数
      * wget http://*.*.*.*/test?timestamp=0\&to=1364966\&direction=0\&page=2\&format=json\&type=5
*   返回空列表对象时，尽可能使用Collections.emptyList();
*   Double,Float处理精度会有问题，使用BigDecimal
*   使用EntrySet遍历HashMap
*   bean拷贝使用cglib中工具类BeanCopier

{% include references.md %}