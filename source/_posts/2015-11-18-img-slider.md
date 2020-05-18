title: 图片轮播nivo-slider插件
layout: post
comments: true
keywords: img_slider
description: img_slider
categories: 
	-html
	-编程记录
date: 2015-11-18 14:50:13
tag: html
---

## 官网资料
[演示地址](http://demo.dev7studios.com/nivo-slider/)
[github地址](https://github.com/gilbitron/Nivo-Slider)
[官方文档](http://docs.dev7studios.com/jquery-plugins/nivo-slider)


## 使用
```
<script src="//cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
<script src="//cdn.bootcss.com/jquery-nivoslider/3.2/jquery.nivo.slider.js"></script>
```
<!-- more -->
```html
<div id="slider" class="nivoSlider" style="text-align: center" >
	<#if imgList??>
		<#list imgList as imgBean>
			<img src="${imgRoot}${imgBean.imgurl}">
		</#list>
	<#else>
		<img src="${root!}/image/banner.jpg">
	</#if>
</div>
```
```
$(window).load(function() {
	$('#slider').nivoSlider({
		slices: 15, // For slice animations
		boxCols: 8, // For box animations
		boxRows: 4, // For box animations
		animSpeed: 500, // Slide transition speed
		pauseTime: 3000, // How long each slide will show
		directionNav:false,
		effect:'slideInLeft',
		controlNav:false,
		pauseOnHover:false // 悬浮是否停止
	});
});
```

## 配置参数
effect: 设置幻灯变化效果，可以使用'random', 或者指定效果如, 'fold,fade,sliceDown'
slices: slice动画效果参数
boxCols: box动画效果参数
boxRows:box动画效果参数
animSpeed: 动画变化速度
pauseTime: 每个幻灯时间
startSlide: 起始幻灯
directionNav: 是否显示“上一个”和“下一个”导航
directionNavHide: 是否悬浮显示导航
controlNav: 是否显示导航控制
controlNavThumbs: 导航是否使用缩略图
controlNavThumbsFromRel: 是否让图片使用rel
controlNavThumbsSearch: 导航缩略图后缀
controlNavThumbsReplace: 导航缩略图替换
keyboardNav: 是否键盘导航
pauseOnHover: 悬浮是否停止
manualAdvance: 是否手动变化幻灯
captionOpacity: 标题透明度
prevText: 上一张文字设定
nextText: 下一张文字设定
randomStart: 是否随机开始
beforeChange: 函数，在幻灯变化前调用
afterChange:函数，在幻灯变化后调用
slideshowEnd:函数，在幻灯变化完成调用
lastSlide:函数，在最后一个幻灯变化完成调用
afterLoad: 函数，slider加载后调用


### effect可配置的参数
sliceDown
sliceDownLeft
sliceUp
sliceUpLeft
sliceUpDown
sliceUpDownLeft
fold
fade
random
slideInRight
slideInLeft
boxRandom
boxRain
boxRainReverse
boxRainGrow
boxRainGrowReverse

