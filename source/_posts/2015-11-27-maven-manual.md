title: maven 常用记录
layout: post
comments: true
keywords: maven\_manual
description: maven\_manual
categories: 使用手册
date: 2015-11-27 09:59:43
tag: 
	- 使用手册
	- maven

---

## 介绍
### 下载地址
[http://maven.apache.org/download.cgi][1]
### 安装方法
[http://maven.apache.org/install.html][2]
### 配置
[http://maven.apache.org/settings.html][3]

<!--more -->

## 常用命令
### 创建一个空项目
```
mvn archetype:generate -DgroupId=com.di.maven -DartifactId=hello-world -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```
### 编译
```
mvn clean 清空生成的文件
mvn validate        验证工程是否正确，所有需要的资源是否可用
mvn compile 编译源代码  
mvn test 编译并测试。
mvn verify 运行任何检查，验证包是否有效且达到质量标准
mvn package 将已编译的源代码打包 (jar或者war)
mvn install 把包安装在本地的repository中，可以被其他工程作为依赖来使用
mvn deploy 在整合或者发布环境下执行，将最终版本的包拷贝到远程仓库，使得其他的开发者或者工程可以共享
mvn package -DskipTests 跳过测试打包项目
mvn dependency:tree -Ddetail=true
```

## pom.xml
```
<project> 根节点
<modelversion> pom.xml 使用的对象模型版本
<groupId> 创建项目的组织或团体的唯一 Id
<artifactId> 项目唯一Id, 项目名
<packaging> 打包扩展名(JAR、WAR、EAR)
<version> 项目版本号
<name> 显示名，用于生成文档
<url> 组织站点，用于生成文档
<description> 项目描述，用于生成文档
<dependency>之<scope> 管理依赖部署
```
### scope
```
compile 缺省值，用于所有阶段，随项目一起发布
provided 期望JDK、容器或使用者提供此依赖。如servlet.jar
runtime 只在运行时使用
test 只在测试时使用，不随项目发布
system 需显式提供本地jar，不在代码仓库中查找
```
## 指定编译的编码
```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-resources-plugin</artifactId>
	<version>2.4</version>
	<configuration>
	    <encoding>UTF-8</encoding>
	</configuration>
</plugin>
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-compiler-plugin</artifactId>
	<version>3.1</version>
	<configuration>
	    <source>1.6</source>
	    <target>1.6</target>
	    <encoding>utf8</encoding>
	</configuration>
</plugin>
```


## 部署到远程仓库

### setting文件位置

settings.xml 的文件可能存在于这2个位子
Maven 的安装目录: $M2\_HOME/conf/settings.xml
操作系统用户目录: ${user.home}/.m2/settings.xml

pom添加
```
<distributionManagement>
	<repository>
	    <id>releases</id>
	    <url>https://test.91umall.com:1443/nexus/content/repositories/releases/</url>
	</repository>
	<snapshotRepository>
	    <id>snapshots</id>
	    <url>https://test.91umall.com:1443/nexus/content/repositories/snapshots/</url>
	</snapshotRepository>
</distributionManagement>
```
setting添加
```
<servers>
	<server>
	    <id>releases</id>
	    <username>admin</username>
	    <password>xxxx</password>
	</server>
	<server>
	    <id>snapshots</id>
	    <username>admin</username>
	    <password>xxxx</password>
	</server>
</servers>
```
## 使用私服仓库
```
 <profiles>
	<profile>
	  <id>uxun</id>
	  <repositories>
	      <repository>
	          <id>uxun</id>
	          <url>https://test.91umall.com:1443/nexus/content/groups/public/</url>
	          <layout>default</layout>
	          <releases>
	            <enabled>true</enabled>
	          </releases>
	          <snapshots>
	            <enabled>true</enabled>
	          </snapshots>
	      </repository>
	  </repositories>
	</profile>
  </profiles>

  <activeProfiles>  
    <activeProfile>uxun</activeProfile>
  </activeProfiles>
  ```

## 部署到中央仓库
[http://www.ttianjun.com/2015/11/10/centermaven/][4]

[1]:	http://maven.apache.org/download.cgi
[2]:	http://maven.apache.org/install.html
[3]:	http://maven.apache.org/settings.html
[4]:	http://www.ttianjun.com/2015/11/10/centermaven/