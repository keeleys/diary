---
title: apache请求过程
date: 2020-05-11 14:05:05
tags:
---

### 一 SpringApplication启动

> 这里只是简短介绍，源码就不贴了，一大块。

1. 构造SpringApplication

> 初始化一些类型和main方法，启动参数，启动的时候会搜索`MATE-INF/spring.factories`下面的配置

{% asset_img 1.png app %}


2. SpringApplication.run

> 核心regist,refresh是spring加载Bean的过程，留着后面spring-context分析。

* 根据执行过程，`触发context监听对应的生命周期方法`
* 根据配置设置一些参数,例如spring.beaninfo.ignore，创建`ConfigurableApplicationContext`
* prepareContext里面构造BeanDefinitionLoader，执行它的load方法，最终执行到spring-context里面的`doRegisterBean`方法
* refreshContext里面最终执行到 spring-context里面的 `refresh()`
