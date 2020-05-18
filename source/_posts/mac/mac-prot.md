title: mac下查看端口是否被占用
tags: mac
categories: mac
date: 2015-06-28 16:53:46
---

### mac下查看端口占用

* 方法一：
> //查看4000端口是否被占用
> sudo lsof -i :4000

* 方法二:
> netstat -anp tcp |grep 4000

* 方法三
> 如下命令可以直接结束占用端口的所有进程：
> lsof -P | grep ':80' | awk '{print $2}' | xargs kill -9
