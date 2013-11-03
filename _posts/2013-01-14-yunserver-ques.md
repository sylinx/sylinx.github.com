---
layout: post
title: 阿里云主机使用笔记-问题记录
category: default
---

* 写磁盘导致load飙得死高

我购买的其中一台云主机（2012.03月份购买）在应用中涉及到写磁盘，一会儿load就飙得非常高，且一段时间都无法降下来。在其它台测试就没有这个问题！ 

问题重现如下，登录主机敲入

    time dd if=/dev/zero of=/test.dbf bs=4k count=100000 
    
load马上飙高，期间主机基本无法响应任何请求,ssh无法操作。且要过一段时间才能恢复 

*解决方法*： 在后台提交工单给阿里云工程师，让他们帮忙改下配置，回复内容如下：

>原因已经找到了由于该机器grub里少配置了一个ide0=noprobe，导致系统盘写数据的时候没有使用pvdriver，影响了IO性能

    
* 不能ssh到外网其他机器上

这个原因可能是自己不小心改错了iptable设置，导致内部无法访问外网80和22端口

*解决方法*：清空重新设置下iptable，具体可参见[阿里云主机使用笔记-环境搭建](http://sylinx.github.com/posts/yunserver-env.html) 中的防火墙设置


{% include references.md %}