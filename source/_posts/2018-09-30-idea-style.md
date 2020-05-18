---
title: Intellij IDEA 代码格式化
date: 2018-09-30 10:18:39
tags: java
---

## 通用代码格式化模板

参考 [格式化模板](https://github.com/vipshop/vjtools/tree/master/standard/formatter)

## 快速注释

idea只有文件头注释,方法的话需要添加一个 Live Templates, 通过关键字来生成方法的注释

![](http://ww1.sinaimg.cn/large/7c8c459dgy1fvritg47kcj21y40yyaiw.jpg)

Template text
```
*
* $VAR1$ 
* <br/>
$params$
* @return $returns$
* @author keeley
* @date $date$ $time$
*/
```

![](http://ww1.sinaimg.cn/large/7c8c459dgy1fvritvh5cfj21kk0q2qas.jpg)
params
```
groovyScript("def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {result+=' * @param ' + params[i] + '\\t' + ((i < params.size() - 1) ? '\\n' : '')}; return result", methodParameters()) 
```

使用的话,在方法上面 输入 `/**` 然后按tab就行了