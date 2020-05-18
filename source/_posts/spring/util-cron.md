title: Cron表达式 and Spring
tags: spring
categories: spring
date: 2015-07-10 13:31:34
---

## 定义
一个cron表达式有至少6个（也可能是7个）由空格分隔的时间元素。从左至右，这些元素的定义如下：  
1．秒（0–59）  
2．分钟（0–59）  
3．小时（0–23）  
4．月份中的日期（1–31）  
5．月份（1–12或JAN–DEC）  
6．星期中的日期（1–7或SUN–SAT）  
7．年份（1970–2099）

<!-- more -->
## 表达式意义   
```
 0 0 10,14,16 * * ?  每天上午10点，下午2点和下午4点   
 0 0,15,30,45 * 1-10 * ?  每月前10天每隔15分钟   
 30 0 0 1 1 ? 2012  在2012年1月1日午夜过30秒时   
 0 0 8-5 ? * MON-FRI  每个工作日的工作时间   
 0 1 0 1 1-12 ? 表示每月1号0点1分执行。  
 0 0 21 ? * 1 表示每个礼拜天21点0分执行。  
 0 0 0 * * ? 表示每天0点0分执行。  
 0 * 22 * * ? 表示每天22点开始每分钟  
 0 * 0-23 * * ? 表示每天每分钟  

 0 * 17 * * ?  每天的从 5:00 PM 至 5:59 PM 中的每分钟触发  
 0 0/5 23 * * ?  每天的从 11:00 PM 至 11:55 PM 中的每五分钟触发 
 0 0/5 15,18 * * ?  每天的从 3:00 至 3:55 PM 和 6:00 PM 至 6:55 PM 之中的每五分钟触发  
 0 0-5 5 * * ?   每天的从 5:00 AM 至 5:05 AM 中的每分钟触发  
 0 0 3 * * ?  每天的 3:00 AM
 0 15 10 ? * MON-FRI  在每个周一,二, 三和周四的 10:15 AM  
 0 15 10 15 * ? 每月15号的 10:15 AM  
 0 15 10 L * ? 每月最后一天的 10:15 AM 
 0 0 12 1/5 * ?  每月从第一天算起每五天的 12:00 PM (中午)  
 0 0 12 * * ?" 每天中午12点触发   
 0 15 10 ? * *" 每天上午10:15触发   
 0 15 10 * * ?" 每天上午10:15触发   
 0 15 10 * * ? *" 每天上午10:15触发   
 0 15 10 * * ? 2005" 2005年的每天上午10:15触发   
 0 * 14 * * ?" 在每天下午2点到下午2:59期间的每1分钟触发   
 0 0/5 14 * * ?" 在每天下午2点到下午2:55期间的每5分钟触发   
 0 0/5 14,18 * * ?" 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发   
 0 0-5 14 * * ?" 在每天下午2点到下午2:05期间的每1分钟触发   
 0 10,44 14 ? 3 WED" 每年三月的星期三的下午2:10和2:44触发   
 0 15 10 ? * MON-FRI" 周一至周五的上午10:15触发   
 0 15 10 15 * ?" 每月15日上午10:15触发   
 0 15 10 L * ?" 每月最后一日的上午10:15触发   
 0 15 10 ? * 6L" 每月的最后一个星期五上午10:15触发   
 0 15 10 ? * 6L 2002-2005" 2002年至2005年的每月的最后一个星期五上午10:15触发   
 0 15 10 ? * 6#3" 每月的第三个星期五上午10:15触发  
 ```

## 顺序和格式

Cron-Expression"由6到7个用空格分开的字段组成的表达式这6或7个字段必须遵循下面的顺  
序和格式：  

| Field Name |  Allowed Values |  Allowed Special Characters  
| :----:|:----:|:----:|
| Seconds  |  0-59   | , - * /  
| Minutes |   0-59  |  , - * /  
| Hours  |  0-23   | , - * /  
| Day-of-month  |  1-31   | , - * ? / L W C  
| Month |   1-12 or JAN-DEC   | , - * /  
| Day-of-Week  |  1-7 or SUN-SAT  |  , - * ? / L C #  
| Year (Optional) |   empty, 1970-2099   | , - * /  

* `*`是一个通配符，表示任何值，用在Minutes字段中表示每分钟。  
* `?`只可以用在day-of-month或者Day-of-Week字段中，用来表示不指定特殊的值。  
* `-`用来表示一个范围，比如10-12用在Month中表示10到12月。  
* `,`用来表示附加的值，比如MON,WED,FRI在day-of-week字段中表示礼拜一和礼拜三和礼拜五。  
* `/`用来表示增量，比如0/15用在Minutes字段中表示从0分开始0和15和30和45分。  
* `L`只可以用在day-of-month或者Day-of-Week字段中，如果用在Day-of-month中，表示某个月  
的最后一天，1月则是表示31号，2月则表示28号（非闰年），如果用在Day-of-Week中表示礼  
拜六（数字7）；但是如果L与数字组合在一起用在Day-of-month中，比如6L，则表示某个月  
的最后一个礼拜六；  

## 定义

### (*)星号  
  
使用星号(*) 指示着你想在这个域上包含所有合法的值。
例如，在月份域上使用星号意味着每个月都会触发这个 trigger。 表达式样例：  
```
0 * 17 * * ?  
```

### ? 问号  
  
? 号只能用在日和周域上，但是不能在这两个域上同时使用。你可以认为 ? 字符是 "我并不关心在该域上是什么值。" 这不同于星号，星号是指示着该域上的每一个值。? 是说不为该域指定值。

### , 逗号  
  
逗号 (,) 是用来在给某个域上指定一个值列表的。例如，使用值 0,15,30,45 在秒域上意味着每15秒触发一个 trigger。  
  
表达式样例： 

```
0 0,15,30,45 * * * ?  
```

### /斜杠  
  
斜杠 (/) 是用于时间表的递增的。我们刚刚用了逗号来表示每15分钟的递增，但是我们也能写成这样 0/15。  
  
表达式样例：  

`0/15 0/30 * * * ? `


意义：在整点和半点时每15秒触发 trigger。


### - 中划线  
  
中划线 (-) 用于指定一个范围。例如，在小时域上的 3-8 意味着 "3,4,5,6,7 和 8 点。"  域的值不允许回卷，所以像 50-10 这样的值是不允许的。  
  
表达式样例：

```
0 45 3-8 ? * *
```
意义：在上午的3点至上午的8点的45分时触发 trigger。

### L 字母  
  
L 说明了某域上允许的最后一个值。它仅被日和周域支持。当用在日域上，表示的是在月域上指定的月份的最后一天。例如，当月域上指定了 JAN时，在日域上的 L 会促使 trigger 在1月31号被触发。假如月域上是 SEP，那么 L 会预示着在9月30号触发。换句话说，就是不管指定了哪个月，都是在相应月份的时最后一天触发 trigger。  
  
表达式 
`0 0 8 L * ?` 
意义是在每个月最后一天的上午 8:00 触发 trigger。在月域上的 * 说明是 "每个月"。  

### W 字母  
  
`W` 字符代表着平日 (Mon-Fri)，并且仅能用于日域中。它用来指定离指定日的最近的一个平日。大部分的商业处理都是基于工作周的，所以 W 字符可能是非常重要的。例如，日域中的 15W 意味着 "离该月15号的最近一个平日。


### (#)井号  
  
(#)字符仅能用于周域中。它用于指定月份中的第几周的哪一天。例如，如果你指定周域的值为 6#3，它意思是某月的第三个周五 (6=星期五，#3意味着月份中的第三周)。


## spring的配置

### xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:task="http://www.springframework.org/schema/task"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
           http://www.springframework.org/schema/task  http://www.springframework.org/schema/task/spring-task-3.0.xsd"
	>
	
	<bean id="htmlTask" class="com.xxx.task.xxxTask" autowire="byType"/> 
		<task:scheduled-tasks>  
        	<task:scheduled ref="htmlTask"  method="process" cron="*/20 * * * * ?" />  
    	</task:scheduled-tasks>
</beans>
```

### java

```java
public class xxxTask {

	public void process(){
		// do something...
	}
	
}
```