title: swift语法概览
description: 'swift,ios'
date: 2015-08-05 21:31:29
tags: 
    - swift
    - ios
categories: ios
---

## 文章
[博客园优秀文章](http://www.cnblogs.com/kenshincui/p/4717450.html)

##简介语法

Playground代码。

```swift

//: Playground - noun: a place where people can play

import UIKit

//1.初始
println("hello world")

//2.常量与变量
var a = "我是变量"
let b = "我是常亮"

//3.指明类型
let letInteger :int_fast32_t = 70;
let letDouble :Double = 70.00
let letString :NSString = "hello"

//4.转换字符串:String()或\()
let myString = "myInt is"
let myInt = 94
let myString2 = myString+String(myInt)
let myString3 = "myInt is\(myInt)"

//5.数组创建与调用
var array = ["one","two","three"]
var getTwo = array[1]

//6.数据字典
var dictionary = ["oneName":"I am one value ","twoName":"I am two value"]
var getTwoValue = dictionary["twoName"]

//7.for 语句
for item in array{
println(item)
}

//8.函数
func getUserName(loginName:String)->String{
return "\(loginName) love Tian"
}

println(getUserName("tt"))

//9.枚举
enum Week{
case 星期一
case 星期二
case 星期三
case 星期四
case 星期五
case 星期六
case 星期日
}

//switch
var today = "星期一"
switch today{
case "星期一" :
println("1");
case "星期二":
println("123")
default:
println("tian")
}

//11.类
class Person:NSObject{
var userName:String;
var userAge = 0;
override init() {
userName="TianJun";
}
}
var p = Person();
p.userAge
```


