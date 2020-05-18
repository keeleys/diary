title: vuejs学习
tags: js
categories: javascript
date: 2015-07-08 09:17:05
---
## 预览
![123](https://dn-tianjun.qbox.me/ttianjunQQ截图20150715145355.png)
## 官网文档
[http://cn.vuejs.org/guide/index.html](http://cn.vuejs.org/guide/index.html)

## 例子 ajax获取数据 加载列表
```html
<div class="content" id="content">
	<span v-text="currentUrl"></span>
	<ul class="img_listbox co">
		<li v-repeat="items" >
			<a href="javascript:;" onclick="prewImage('{{imgUrl}}')">
			<img v-attr="src:'${imgRoot}'+imgUrl"/>
			</a>
			<p><span v-text="upTime"></span></p>
			<p><a class="btn btn-success" style="display: inline;padding: 6px 12px;" role="button" href="${root}/user/qrcode?imgID={{imgID}}">打印</a></p>
		</li>
	</ul>
	<a class="more" href="javascript:;" onclick="loadDate()" v-show="moreflag">查看更多</a>
</div>
```
```html
<script src="//cdnjs.cloudflare.com/ajax/libs/vue/0.12.16/vue.min.js"></script>
<script>
var pageNum = 1;
var vm = new Vue({
  el: '#content',
  data: {
	 items: [],
	 moreflag:true
  },
 compiled:loadDate
});
function loadDate(){
	$.post(root+"/user/imglistJson/"+pageNum,function(data){
		if(data.list.length==0){
			vm.moreflag=false;
			return;
		}
		vm.items=vm.items.concat(data.list);
		pageNum++;
	},"json")
}
</script>
```