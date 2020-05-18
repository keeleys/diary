title: 收集整理的一些java工具类
layout: post
comments: true
keywords: common-kit
description: common-kit
categories: java
date: 2015-10-21 15:32:09
tag: java
---

[项目地址](https://github.com/keeleys/common-kit)

## 基础工具类

`GenerateSequenceUtil` 根据时间生成唯一标识，每秒钟9999内唯一

`Lists` 生成list map, 泛型复杂的时候比较好用

`PathKit` 获取文件路径的工具类，提取自jfinal

`PropKit`,`Prop` 读取配置文件的工具类，2个组合使用。
```
//可以加载多个 但第二个文件开始需要每次都use例如
public static final Prop weixinProp = PropKit.use("weixin.txt");
public static final String APPID =weixinProp.get("appid");
public static final String SECRET =weixinProp.get("secret");
```

`ClassSearcher`  类搜索,jar包搜索,注解反射用

## 文件操作

`PoiExporter` Excel导出的工具类 

`ImageMarkLogoUtil` 图片水印工具类 

`ImageKit` 图片裁剪,缩放,拼贴


## 网络操作

`HttpKit` http访问类，get,post请求

## json xml解析
`JsonUtil`,`XmlUtils`,`XJDataNode` 三个组合 通用解析.



## 文件加密
`Base64Utils`,`MD5Util` base64,md5加密
`DecriptUtil` SHA1加密(如微信)

`StrKit` 字符串工具类 ,判断空，首字母小写，等

## 微信开发工具类
`WeixinManager` 获取token jsapi签名等
`CacheKit` ehcache 工具类
## 测试工具类

`BaseTest` webapp环境的注解测试父类,依托spring-test
