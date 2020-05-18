---
title: ubuntu 环境配置
date: 2016-04-01 21:54:58
tags: linux
toc : true
---

## python环境

### pip
第一步是安装pip,后面很多软件需要用pip安装，有它比较方便
```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

### 编码设置 
有可能会遇到编码错误，可以运行`locales`查看你的默认编码 安装缺失的语言包
```bash
# LANG=en_US.UTF-8
apt-get install language-pack-en-base
export LC_CTYPE="en_US.UTF-8"
dpkg-reconfigure locales

# LANG=zh_CN.UTF-8
apt-get install language-pack-zh-hans
dpkg-reconfigure locales
```

<!-- more -->
### lxml
安装lxml,很多包都依赖它。优先安装它,因为它安装过程非常占用内存
```bash
# 安装依赖
apt-get install libxml2-dev libxslt1-dev python-dev
# 大内存方案
pip install lxml
# 小内存方案 小内存运行上面那行基本不成功。。
apt-get install python-lxml
```

## linux 定时任务
> 需要注意的是 定时任务会根据当前用户的默认目录`~`开始执行脚本或者任务的，所以如果脚本有上下文环境的，进`~`目录先测试运行一次

```bash
# 用cron表达式控制任务的执行周期
crontab -e
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
```
```bash
# 查看cron服务的运行状态
ps aux | grep cron # 查看cron进程
service cron status # 查看服务状态
```
```bash
#启动和关闭
service cron restart # 重启
service cron reload #重新加载配置
service cron stop #停止定时任务服务
```

## 一些应用
### mysql
安装mysql
```bash
# ubuntu的apt-get很方便，当然也可以用yum或者在官网下源包
apt-get install mysql-server
#如果需要安装python
apt-get install libmysqld-dev 
# 导入
source /root/data/git/funny2/doc/funny.sql


```
### git
```bash
# 这个没啥好说的 
apt-get install git
# 配置ssh,运行下面这个，然后一路按`回车键`，不设置密码
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# 最后在你的目录下面会生出密钥 `~/.ssh/id_rsa.pub`
```

### 安装zsh(一种 Shell 内核，替换bash)
[http://ohmyz.sh](http://ohmyz.sh)
```bash
apt-get zsh
# 在安装oh-my-zsh
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```
autojump
```bash
# 安装 autojump 它能自动找到最常访问的目录，补全目录
apt-get install autojump
# 将autojump添加到zsh插件里面去
vi ~/.zshrc
# 修改 plugins 
plugins=(git autojump github)
```
autojump 使用方法
```bash   
j fun  # j是简写，fun是以前cd过的目录的一部分
/root/data/git/funny2
```


## 一些配置
### mysql配置文件
```bash
vi /etc/mysql/my.cnf
```

```conf
#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
#
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.
[client]
port    = 3306
socket    = /var/run/mysqld/mysqld.sock

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

# This was formally known as [safe_mysqld]. Both versions are currently parsed.
[mysqld_safe]
socket    = /var/run/mysqld/mysqld.sock
nice    = 0

[mysqld]
#
# * Basic Settings
#
user    = mysql
pid-file  = /var/run/mysqld/mysqld.pid
socket    = /var/run/mysqld/mysqld.sock
port    = 3306
basedir   = /usr
datadir   = /var/lib/mysql
tmpdir    = /tmp
lc-messages-dir = /usr/share/mysql
skip-external-locking
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address    = 127.0.0.1
#
# * Fine Tuning
#
key_buffer    = 16M
max_allowed_packet  = 16M
thread_stack    = 192K
thread_cache_size       = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover         = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10
#
# * Query Cache Configuration
#
query_cache_limit = 1M
query_cache_size        = 16M
#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file        = /var/log/mysql/mysql.log
#general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Here you can see queries with especially long duration
#log_slow_queries = /var/log/mysql/mysql-slow.log
#long_query_time = 2
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
#server-id    = 1
#log_bin      = /var/log/mysql/mysql-bin.log
expire_logs_days  = 10
max_binlog_size         = 100M
#binlog_do_db   = include_database_name
#binlog_ignore_db = include_database_name
#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
#
# * Security Features
#
# Read the manual, too, if you want chroot!
# chroot = /var/lib/mysql/
#
# For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
#
# ssl-ca=/etc/mysql/cacert.pem
# ssl-cert=/etc/mysql/server-cert.pem
# ssl-key=/etc/mysql/server-key.pem



[mysqldump]
quick
quote-names
max_allowed_packet  = 16M

[mysql]
#no-auto-rehash # faster start of mysql but no tab completition

[isamchk]
key_buffer    = 16M

#
# * IMPORTANT: Additional settings that can override those from this file!
#   The files must end with '.cnf', otherwise they'll be ignored.
#
!includedir /etc/mysql/conf.d/
```


 