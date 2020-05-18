title: 使用beetl生成mybatis的mapping文件
tags: 
 - beetl
 - mybatis
 - 注解
categories: 
 - beetl
 - mybatis
date: 2015-07-20 14:37:32
---
## 使用前因

由于项目历史原因，数据库字段都是中文命名的，所以做了这个小工具，还很多不完善的地方，比如说注解类略多，生成方法较少，处理类型单一等。

## 项目结构

![项目结构](https://dn-tianjun.qbox.me/ttianjunQQ截图20150720145701.png)

<!-- more -->

## beetl 模版

号称最快的模版，用起来也比较方便，可以自定义模版语法的“界限”。
[官网在线体验](http://ibeetl.com:8080/beetlonline/)

## 实体和table映射关系

 > 由于都是中文，中文到英文的映射没有什么具体规则好自动实现，也不太想用拼音，然后在用的过程中写javabean的时候都要写注释标明这行对应的数据库哪个字段，后来想想何不让注解更有意义点呢，然后就用了注解实现.

### 数据库表名
```
/**
 * 表名
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface KTable {
    String value();
}

```

### 表主键
```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface KId {
    String value() default "";
}
```

### 表字段
```
/**
 * 数据库属性对应列
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface KField {
    String value() default "";
}
```

### 表查询
```
/**
 * 界面查询列
 * Created by TianJun on 2015-05-29.
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface KParam {
    String value() default "";
}
```

## beetl配置

### maven
```
 <dependencies>
    <dependency>
        <groupId>org.beetl</groupId>
        <artifactId>beetl-core</artifactId>
        <version>2.2.2</version>
    </dependency>
</dependencies>
```

### beetl.properties
beetl配置文件,默认从classpath读取,放到resources根目录,内容如下
```
-- 设置起始结束标签 默认是 <% 和 %>
DELIMITER_STATEMENT_START=@
DELIMITER_STATEMENT_END=
```

### 定义要生成的模版
直接把map.xml文件复制下来改改，把可以动态的改成动态,只做了单表的update,select,insert操作和resultMap对应操作.

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="${className}Dao" >
<resultMap type="${className}" id="resultMap">
    @if(!isEmpty(tableId)){
    <id property="${tableId.field}" column="${tableId.column}" jdbcType="VARCHAR" />
    @}
    @for(user in fields){
    <result property="${user.key}" column="${user.value}" jdbcType="VARCHAR" />
    @}
</resultMap>

<select id="list" resultMap="resultMap">
	select * from ${tableName} where 1=1
	@for(p in params){
	@if(strutil.endWith(p.key,"Date")||strutil.endWith(p.key,"Time")){
    <if test="beginTime!=null and beginTime!=''">
        and to_char(${p.value},'yyyy-MM-dd hh24:mi:ss') <![CDATA[ >= ]]>  #{beginTime}
    </if>
    <if test="endTime!=null and endTime!=''">
        and to_char(${p.value},'yyyy-MM-dd hh24:mi:ss') <![CDATA[ <= ]]>  #{endTime}
    </if>
    @}else{
    <if test="${p.key}!=null and ${p.key}!=''">and ${p.value} = #{${p.key}}</if>
    @}
    @}
</select>

<insert id="insert" parameterType="${className}">
    insert into ${tableName}
    <trim prefix="(" suffix=")" suffixOverrides=",">
        @if(!isEmpty(tableId)){
        ${tableId.column},
        @}
        @for(field in fields){
        <if test="${field.key}!=null and ${field.key}!=''">${field.value},</if>
        @}
    </trim>
    <trim prefix="values(" suffix=")" suffixOverrides=",">
        @if(!isEmpty(tableId)){
        #{${tableId.field}},
        @}
        @for(field in fields){
        @if(strutil.endWith(field.key,"Date")||strutil.endWith(field.key,"Time")){
        <if test="${field.key}!=null and ${field.key}!=''">to_date(#{${field.key}},'yyyy-mm-dd hh24:mi:ss'), </if>
        @}else{
        <if test="${field.key}!=null and ${field.key}!=''"> #{${field.key}},</if>
        @}
        @}
    </trim>
</insert>

<update id="update">
    update  ${tableName} set
    <trim   suffixOverrides="," >
        @for(field in fields){
        @if(strutil.endWith(field.key,"Date")||strutil.endWith(field.key,"Time")){
       <if test="${field.key}!=null and ${field.key}!=''"> ${field.value}=to_date(#{${field.key}},'yyyy-mm-dd hh24:mi:ss'),</if>
        @}else{
       <if test="${field.key}!=null and ${field.key}!=''"> ${field.value}=#{${field.key}},</if>
        @}
        @}
    </trim>
    where ${tableId.column!id} = #{${tableId.field!}}
</update>
</mapper>
```

## 生成工具类

### MyBatisUtil.java
```java
package com.ttianjun.mybatis;

import com.ttianjun.mybatis.test.*;
import org.beetl.core.Configuration;
import org.beetl.core.GroupTemplate;
import org.beetl.core.Template;
import org.beetl.core.resource.ClasspathResourceLoader;

import java.io.IOException;
import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * Created by wodetianjun on 15/5/27.
 */
public class MyBatisUtil {
    public static void main(String [] args) throws IOException {
        String xml=MyBatisUtil.createXml(Message.class);
        System.out.println(xml);//直接把结果打印到控制台了，可以改成输入到文件或者网页.
    }

    public static String createXml(Class c) throws IOException {
        ClasspathResourceLoader resourceLoader = new ClasspathResourceLoader();
        Configuration cfg = Configuration.defaultConfiguration();
        GroupTemplate gt = new GroupTemplate(resourceLoader, cfg);//从classpath资源加载器获取beetl配置文件
        Template t = gt.getTemplate("/template.txt"); //获取模版

        String tableName = c.getSimpleName(); //如果类上没有@KTable注解 默认的名字就是类名
        KTable table = (KTable) c.getAnnotation(KTable.class);
        if(table!=null) tableName = table.value();

        Map<String,String> idBean = buildId(c);
        t.binding("className", c.getCanonicalName()); //t.binding 给模版绑定数据来源
        t.binding("fields", buildFields(c));
        t.binding("params", buildParam(c));
        t.binding("tableName",tableName);
        t.binding("tableId",buildId(c));
        return t.render(); //生成输入
    }
    //生成id的map  设想的ID只有一个 还不支持多列ID
    private static Map<String,String> buildId(Class bean){
        Map<String,String> map = new LinkedHashMap<String, String>();
        Field[] fields=bean.getDeclaredFields(); 获取所有字段
        for(Field field : fields){
            String key = field.getName();
            KId id=field.getAnnotation(KId.class);//判断字段是否有这个注解
            if(id!=null) {
                String fieldName = field.getName();
                String value = id.value().isEmpty() ? fieldName : id.value();
                map.put("field", fieldName);
                map.put("column", value);
                return map;
            }
        }
       
    }
    private static Map<String,String> buildFields(Class bean){
        Map<String,String> map = new LinkedHashMap<String, String>();
        Field[] fields=bean.getDeclaredFields();
        for(Field field : fields){
            String key = field.getName();
            KField commit=field.getAnnotation(KField.class);
            if(commit!=null) {
                String value = commit.value().isEmpty() ? field.getName() : commit.value();
                map.put(key, value);
            }
        }
        return map;
    }
    private static Map<String,String> buildParam(Class bean){
        Map<String,String> map = new LinkedHashMap<String, String>();
        Field[] fields=bean.getDeclaredFields();
        for(Field field : fields){
            String key = field.getName();
            KParam commit=field.getAnnotation(KParam.class);
            if(commit!=null) {
                String value = commit.value().isEmpty() ? field.getName() : commit.value();
                map.put(key, value);
            }
        }
        return map;
    }
}

```


## 使用

### 写一个bean 用到上面的注解
```
public class PrepaidDetail {
    @KId("记录编号")
    private String recordNo;//记录编号

    @KField("客户编号")
    private String custNo;//客户编号
    @KField("来源")
    private String source;//来源
    @KField("金额")
    private String amount;//金额
    @KField("使用的转运商单号")
    private String transNo;//使用的转运商单号

    @KParam("操作时间")
    @KField("操作时间")
    private String optTime;//操作时间
    @KField("币种")
    private String moneyType;//币种
    @KField("备注")
    private String remark;//备注
    @KParam("有效标识")
    @KField("有效标识")
    private String invalid;//有效标识;
    @KField("支付交易号")
    private String tradeCode;//支付交易号
}

```
### 生成

```
 String xml=MyBatisUtil.createXml(PrepaidDetail.class); //传入对应的class生成
```

## github

* github地址 [https://github.com/keeleys/mybatis_template](https://github.com/keeleys/mybatis_template)

