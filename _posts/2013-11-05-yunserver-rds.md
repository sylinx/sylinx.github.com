---
layout: post
title: 阿里云主机使用笔记-RDS备份恢复(4)
category: topic
---

1.取得mysql root权限

    update user set password = Password('root') where User = 'root';
    update user set Host = '%' where user = 'root';
    FLUSH PRIVILEGES;

2.从RDS下载数据到本地目录


3.解压
tar vizxf hins29776_xtra_20130915035603.tar.gz


4.执行数据恢复

    innobackupex-1.5.1 --defaults-file=/etc/my.cnf  --user=root --password=root  /root/data
    innobackupex-1.5.1 --defaults-file=/etc/my.cnf --user=root --password=root --copy-back  /root/data
    innobackupex-1.5.1 --defaults-file=/etc/my.cnf  --user=root --password=sylinx  /root/data
    innobackupex-1.5.1 --user=root --password=sylinx --copy-back  /root/data


{% include references.md %}