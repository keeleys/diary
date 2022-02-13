---
title: 第三方系统对接的几个考虑点
date: 2020-05-11 14:05:05
tags: 设计相关
---

>> 一些经验总结，先列个大纲，具体例子再补充。

1. 状态设计：登录状态公共管理，只有一处处理，减少重复的验证请求 （例如微信带token等，都是要全局缓存唯一的）
2. 异常设计：例如登录失效自动注销缓存，重复提交或者已经存在的异常等错误解析成不同错误code封装成异常抛出,异常越精细系统越健壮
3. 频次设计：第三方接口能承受的调用频次，在调用的时候做限流，数据同步方面设计的时候考虑到频次
4. 日志设计：调用前调用后的日志和参数都打印记录起来，方便排查第三方问题的时候能提供确实的参数复现。
5. 幕等设计：如果系统出错，重复调用第三方的时候，能保证一次调用和多次调用都是一样的结果。有唯一的业务编号和第三方业务编号设计。
6. 补偿设计：通过消息队列，分布式定时任务等，补偿失败的数据，重要的失败数据还需要落库保存人工处理
7. 网络设计：网络通畅要得到保证