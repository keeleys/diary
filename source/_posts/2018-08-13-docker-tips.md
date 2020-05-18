---
title: docker-tips
date: 2018-08-13 11:35:05
tags:
---

写dockerfile文件
然后build -t title .
然后

## docker 删除none镜像
```sh
docker stop $(docker ps -a | grep "Exited" | awk '{print $1 }')   //停止容器  
docker rm $(docker ps -a | grep "Exited" | awk '{print $1 }')    //删除容器  
docker rmi $(docker images | grep "none" | awk '{print $3}')    //删除镜像
```
<!-- more -->