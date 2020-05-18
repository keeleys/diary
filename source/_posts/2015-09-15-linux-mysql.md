title: 阿里云主机安装myslq5.5和tomcat 7
layout: post
comments: true
keywords: linux-mysql
description: linux-mysql
categories: linux
date: 2015-09-15 14:55:32
tag: linux
---

## 环境
>先连接到内网跳板机器，然后通过内网跳板机连接阿里云

### linux
```
rpm -q centos-release 
#centos-release-6-3.el6.centos.9.x86_64

cat /etc/issue

```
### 查看mysql 是否安装
```
rpm -qa | grep mysql
```
### 卸载旧版本的mysql
```
sudo yum -y remove mysql-xx-xx
```

##  步骤

###  连接跳板机
```
ssh jfy@test.xxx.com -p 6021
```
### 下载mysql到当前目录

目前5.5的最新版5.5.45，官网地址 [http://dev.mysql.com/downloads/mysql/#downloads](http://dev.mysql.com/downloads/mysql/#downloads)

```
wget -c http://dev.mysql.com/get/Downloads/MySQL-5.5/MySQL-server-5.5.45-1.el6.x86_64.rpm
wget -c http://dev.mysql.com/get/Downloads/MySQL-5.5/MySQL-client-5.5.45-1.el6.x86_64.rpm
wget -c http://dev.mysql.com/get/Downloads/MySQL-5.5/MySQL-devel-5.5.45-1.el6.x86_64.rpm
```
### 传输到阿里云服务器

```
scp -r . root@10.xxx.xxx.xxx:/home
```
### 连接阿里云服务器
```
ssh jfy@10.xxx.xx.xx-p 22
```
### 用rpm命令安装
```
rpm -ivh MySQL-server-5.5.45-1.el6.x86_64.rpm
rpm -ivh MySQL-client-5.5.45-1.el6.x86_64.rpm
rpm -ivh MySQL-devel-5.5.45-1.el6.x86_64.rpm
```

### 修改root密码
```
/usr/bin/mysqladmin -u root password '123456'
```

###  启动 停止mysql
```
service mysql start
service mysql stop
```

### 登陆mysql

```
mysql -u root -p
```

### 查看mysql安装目录
```
whereis mysql
```

### 下载tomcat7
>地址http://tomcat.apache.org/download-70.cgi

```
wget -c http://mirrors.hust.edu.cn/apache/tomcat/tomcat-7/v7.0.64/bin/apache-tomcat-7.0.64.tar.gz
```

### 解压 然后根据bin目录下面的命令运行服务

```
tar -zxvf apache-tomcat-7.0.64.tar.gz

```

## 假如是el5的服务器
```
下载这3个安装
http://cdn.mysql.com//Downloads/MySQL-5.5/MySQL-server-5.5.46-1.rhel5.x86_64.rpm
http://cdn.mysql.com//Downloads/MySQL-5.5/MySQL-client-5.5.46-1.rhel5.x86_64.rpm
http://cdn.mysql.com//Downloads/MySQL-5.5/MySQL-devel-5.5.46-1.rhel5.x86_64.rpm
```
### mysql 远程连接
```
update user set host='%' where user='root' and host='localhost';
flush privileges;        #刷新权限表，使配置生效
```
如果您想关闭远程连接，恢复mysql的默认设置（只能本地连接），您可以通过以下步骤操作：
```
use mysql                #打开mysql数据库

#将host设置为localhost表示只能本地连接mysql

update user set host='localhost' where user='root';

flush privileges;        #刷新权限表，使配置生效
```
您也可以添加一个用户名为yuancheng，密码为123456，权限为%（表示任意ip都能连接）的远程连接用户。命令参考如下：
```
grant all on *.* to 'yuancheng'@'%' identified by '123456';

flush privileges;
```