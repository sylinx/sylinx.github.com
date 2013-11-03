---
layout: post
title: 阿里云主机使用笔记-环境搭建
category: default
---
申请开通下云主机，我这里操作系统选择的是redhat5.4 64位。我们系统都是用java编写，所以web服务器和应用容器采用 nginx+jetty。 常用软件均安装在/usr/soft/install下,链接到/usr/soft/,这样以后更新install目录下的软件版本，只要更新下链接就可。

系统安装初始化后，需要设置下更新源，在/etc/yum.repos.d/ 下新建 CentOS-Base.repo\([具体内容见此](https://github.com/sylinx/shutils/blob/master/CentOS-Base.repo)\),然后执行

     yum update


<h3>防火墙配置</h3>

清空和设置防火墙脚本如下：

[/usr/soft/sh/flush-iptables](https://github.com/sylinx/shutils/blob/master/flush-iptables) (清空)  
[/usr/soft/sh/setup-iptables](https://github.com/sylinx/shutils/blob/master/setup-iptables)（设置防火墙规则）

执行以上两个脚本后，/etc/rc.d/init.d/iptables save 保存下iptables配置。启动和关闭防火墙

    service iptables start
    service iptables stop
    service iptables status




<h3>安装常用软件</h3>

可以通过以下命令查找并安装常用软件，如安装lftp

    yum list|grep lftp
    yum install lftp.x86_64

常见的zip、unzip装上

<h3>安装nginx</h3>

安装编译环境

    yum -y install gcc automake autoconf libtool make
    yum -y install pcre-devel openssl openssl-devel
    
编译nginx

    cd /usr/soft/install
    wget http://nginx.org/download/nginx-1.0.14.tar.gz
    tar xvf nginx-1.0.14.tar.gz
    cd nginx-1.0.14
    ./configure --prefix=/usr/soft/install/nginx
    make  & make install 
   
建立软链接

    ln -s /usr/soft/install/nginx /usr/soft/nginx
  
nginx80端口非ROOT用户授权
    
    chmod u+s /usr/soft/nginx/sbin/nginx
    
测试nginx

    /usr/soft/nginx/sbin/nginx -t -c /usr/soft/nginx/conf/nginx.conf
    
<h3>安装java</h3>

到http://www.oracle.com/technetwork/java/javase/downloads/jdk-6u31-download-
1501634.html 下载 jdk-6u31-linux-x64-rpm.bin，放到/usr/soft/install下，然后执行./jdk-6u31-linux-x64-rpm.bin

建立软链接

    ln -s /usr/java/jdk1.6.0_31 /usr/soft/java
    
测试是否正常安装

    /usr/soft/java/bin/java -version
 
<h3>安装jetty</h3>

下载JETTY

    wget http://repo1.maven.org/maven2/org/eclipse/jetty/jetty-distribution/7.2.2.v20101205/
    jetty-distribution-7.2.2.v20101205.tar.gz  

直接解压安装

    tar xvf jetty-distribution-7.2.2.v20101205.tar.gz    
    ln -s /usr/soft/install/jetty-distribution-7.2.2.v20101205 /usr/soft/jetty

<h3>解决TIME_WAIT过多问题</h3>

nginx代理使用短链接的方式和后端jetty交互，会使TIME_WAIT变得比较多,解决方法如下：


编辑/etc/sysctl.conf文件，加入以下内容：
>net.ipv4.tcp_syncookies = 1  
net.ipv4.tcp\_tw\_reuse = 1  
net.ipv4.tcp\_tw\_recycle = 1  
net.ipv4.tcp\_fin\_timeout = 30  
\# Decrease the time default value for tcp\_keepalive\_time connection  
net.ipv4.tcp\_keepalive\_time = 1800  
\# Turn off tcp\_window\_scaling  
net.ipv4.tc\p_window\_scaling = 0  
\# Turn off the tcp\_sack  
net.ipv4.tcp\_sack = 0  
\#Turn off tcp\_timestamps  
net.ipv4.tcp\_timestamps = 0  
vm.swappiness = 10  

然后执行 /sbin/sysctl -p 让参数生效,重启下网络 service network restart

另外需要在 /etc/rc.d/rc.local里添加已使得重启的时候生效
>echo "30">/proc/sys/net/ipv4/tcp_fin_timeout  
echo "1800">/proc/sys/net/ipv4/tcp_keepalive_time  
echo "0">/proc/sys/net/ipv4/tcp_window_scaling  
echo "0">/proc/sys/net/ipv4/tcp_sack  
echo "0">/proc/sys/net/ipv4/tcp_timestamps  

<h3>设置limits.conf</h3>

vi /etc/security/limits.conf, 增加或者放开修改如下：

>\* \- nofile 10240

<h3>同步时间</h3>

我购买那会儿较早，云主机服务器时间同步有问题，经常跑得比较快，因此用如下脚本手工或者crontab定时同步

    service ntpd stop
    ntpdate ntp.api.bz



{% include references.md %}