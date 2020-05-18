title: 给网站添加友好的bootcss样式提示
tags: html
categories: html
date: 2015-06-29 11:41:08
---

## 界面
![界面](https://dn-tianjun.qbox.me/ttianjunQQ截图20150715144649.png)

## 要求

| Min.Bootstrap version | Max.Bootstrap | Min.jQuery
| ----|----|----|
| 3.0.0 | 3.x.x | 1.9.1|

<!-- more -->

## 安装
* npm
```
npm install bootbox
```

## 用法
* 在jquery bootstrap.js 后面，引入bootbox.min.js 文件
* 使用前的统一的配置
```
bootbox.setDefaults({
	//size: 'small',
	locale:"zh_CN",
	title:'温馨提示'
});
```

* 页面使用
```
bootbox.alert("Your message here…");
bootbox.alert("Your message here…", function(){ /* your callback code */ });
bootbox.confirm("Your message here…", function(result){ /* your callback code */ });
bootbox.prompt("Your message here…", function(result){ /* your callback code */ });
```
<!-- more -->
## 官网文档参考
> [http://bootboxjs.com/documentation.html](http://bootboxjs.com/documentation.html)
