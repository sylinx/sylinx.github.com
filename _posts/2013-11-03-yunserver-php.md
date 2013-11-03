---
layout: post
title: 阿里云主机使用笔记-php环境搭建
category: default
---
<h3>安装依赖包</h3>
    yum -y install gcc automake autoconf libtool make
 
    yum -y install gcc gcc-c++ glibc
 
    yum -y install libmcrypt-devel mhash-devel libxslt-devel \
    libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel     libxml2 libxml2-devel \
    zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel \
    ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel \
    krb5 krb5-devel libidn libidn-devel openssl openssl-devel

<h3>php-fpm安装</h3>
    wget http://cn2.php.net/distributions/php-5.4.7.tar.gz
    tar zvxf php-5.4.7.tar.gz
    cd php-5.4.7
    ./configure --prefix=/usr/local/php  --enable-fpm --with-mcrypt \
    --enable-mbstring --disable-pdo --with-curl --disable-debug  --disable-rpath \
    --enable-inline-optimization --with-bz2  --with-zlib --enable-sockets \
    --enable-sysvsem --enable-sysvshm --enable-pcntl --enable-mbregex \
    --with-mhash --enable-zip --with-pcre-regex --with-mysql --with-mysqli \
    --with-gd --with-jpeg-dir
    make all install

如果中途出现"virtual memory exhausted: Cannot allocate memory"错误，请加大内存

<h3>php-fpm启停</h3>
    #测试php-fpm配置
    /usr/local/php/sbin/php-fpm -t
    /usr/local/php/sbin/php-fpm -c /usr/local/php/etc/php.ini -y /usr/local/php/etc/php-fpm.conf -t
 
    #启动php-fpm
    /usr/local/php/sbin/php-fpm
    /usr/local/php/sbin/php-fpm -c /usr/local/php/etc/php.ini -y /usr/local/php/etc/php-fpm.conf
 
    #关闭php-fpm
    kill -INT `cat /usr/local/php/var/run/php-fpm.pid`
 
    #重启php-fpm
    kill -USR2 `cat /usr/local/php/var/run/php-fpm.pid`

<h3>nginx配置</h3>


如果连接mysql出现以下错误
"mysqlnd cannot connect to MySQL 4.1+ using the old insecure authentication"

可以用如下方法解决

修改/etc/my.cnf ，将old_passwords=1改成 old_passwords=0

执行以下更新操作：

    UPDATE mysql.user SET Password = PASSWORD('sylinx') WHERE user = 'root'; 
    FLUSH PRIVILEGES; 



{% include references.md %}