title: dubbo介绍和集成例子
layout: post
comments: true
keywords: dubbo
categories: 分布式
date: 2016-01-14 11:05:36
tag: rpc
---
## dubbo介绍

>DUBBO是一个分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，是阿里巴巴SOA服务化治理方案的核心框架
 
 一句话概括，就是将服务层抽出来，拆成一个个单独的服务，然后通过远程调用供前置使用层调用。通过dubbo封装之后，调用远程方法和调用本地方法一样的方便
<!-- more -->

* 官网[http://dubbo.io](http://dubbo.io)

* 架构图
![http://dubbo.io/dubbo-architecture.jpg-version=1&modificationDate=1330892870000.jpg](http://dubbo.io/dubbo-architecture.jpg-version=1&modificationDate=1330892870000.jpg)

## 集成步骤(依托spring集成)
* 下载[注册中心zookeeper](http://mirrors.hust.edu.cn/apache/zookeeper/)，解压，然后运行bin/zkService，供提供者和使用者发布服务
* 下载[dubbo-admin](https://github.com/keeleys/dubbo-demo/tree/master/file),服务调度和管理中心，能查看和管理服务(非必需),(默认登陆的账号密码root,`root`) 
* 编写提供服务的api 和 api可能用到的实体
* 编写服务，并运行服务
* 编写调用着，选择服务，然后调用

## github 例子项目

[https://github.com/keeleys/dubbo-demo](https://github.com/keeleys/dubbo-demo)
