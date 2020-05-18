title: javascript 常用操作
tags: js
categories: 前端
description: javascript 常用操作
date: 2015-06-27 14:33:11
---


## 字符转Json对象

```javascript
var obj = eval('(' + str + ')');
```

## 刷新页面

```
window.location.reload(true);  
window.location.href="/blog/window.location.href";   
window.navigate("url");   
```

## 定时执行

```js
setTimeout("abc()",10000); 
setInterval("function",time) ;
```

## 控制只能输入数字和小数点

```js
onkeypress="return (/[\d.]/.test(String.fromCharCode(event.keyCode)))"
```