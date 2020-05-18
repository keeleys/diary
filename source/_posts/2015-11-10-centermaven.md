title: 发布构建到中央仓库
layout: post
comments: true
keywords: centermaven
description: centermaven
categories: maven
date: 2015-11-10 14:21:45
tag: maven
---


> 今天把一个maven项目发布到中央仓库了 网上教程很多 所以就不详细描述了，简略记录一下过程

* 参考资料[http://my.oschina.net/huangyong/blog/226738#OSC_h3_5](http://my.oschina.net/huangyong/blog/226738#OSC_h3_5)
* 项目地址[https://github.com/keeleys/common-kit](https://github.com/keeleys/common-kit)

<!-- more -->

## 发布构建到中央仓库

1. [注册](https://issues.sonatype.org/secure/Signup!default.jspa)
2. [提交申请](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134) 
3. 下载[Gpg4win-Vanilla](https://www.baidu.com/link?url=eQdA7UsQr1K1mSbLTJLz47rOTrdshxcw7hxanLwsdd4Z-Fc2QUfydZ1rtZV47RbP8WXg_tH2gkjgupLGxRKPmq&wd=&eqid=8f90d70a000094c40000000556418db1) 
```
gpg --version  查看是否安装成功
gpg --gen-key  生成GPG文件
前面可以一路回车，注意填写realname,email就好了，注意 Passphase的地方输入密码
gpg --list-keys 查看公钥 pub   2048R/xxx xxx是密钥
```

4. 将公钥发布到 PGP 密钥服务器
```
发布的时候可能出错， 浏览器访问下 sks-keyservers.net看看是否能正常访问
gpg --keyserver hkp://pool.sks-keyservers.net --send-keys xxx 发布
gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys xxx 查询发布
```

mac 

```
brew install gpg

> gpg --version
gpg (GnuPG) 1.4.20
Copyright (C) 2015 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

```

5. 修改setting.xml 添加server
```xml
<servers>
    <server>
        <id>oss</id>
        <username>用户名</username>
        <password>密码</password>
    </server>
</servers>
```

6. pom.xml修改 [参考](https://github.com/keeleys/common-kit/blob/master/pom.xml)

7. 发布项目
```
mvn clean deploy -Dgpg.passphrase=你的密码
```

8. 在 OSS 中[发布构件](https://oss.sonatype.org/),登陆之后，点击`Staging Repositories` 查看自己的构件，然后点击`Close `,然后点击`Release `.如果出错，构建上面会有数字错误提示，按照提示修改再重新操作.

9. 在曾经创建的 [Issue](https://issues.sonatype.org/secure/ViewProfile.jspa) 下面回复一条“构件已成功发布”的评论

10. 中央仓库搜索网站：[http://search.maven.org/](http://search.maven.org/)

11. 以上都成功之后，以后同groupid的只构建 只需要第 6,7,8这几步咯。上传之后要等几小时中央仓库同步


## pom.xml 

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.ttianjun</groupId>
	<artifactId>common-kit</artifactId>
	<version>1.0.1</version>
	<parent>
		<groupId>org.sonatype.oss</groupId>
		<artifactId>oss-parent</artifactId>
		<version>7</version>
	</parent>
	<name>common-kit</name>
	<description>java kit </description>
	<url>https://github.com/keeleys/common-kit</url>
	<licenses>
		<license>
			<name>The Apache Software License, Version 2.0</name>
			<url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
		</license>
	</licenses>
	<developers>
		<developer>
			<name>keeley</name>
			<email>keeley.jun@qq.com</email>
		</developer>
	</developers>
	<scm>
		<connection>scm:git:https://github.com/keeleys/common-kit.git</connection>
		<developerConnection>scm:git:https://github.com/keeleys/common-kit.git</developerConnection>
		<url>https://github.com/keeleys/common-kit.git</url>
	</scm>
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.0</version>
				<configuration>
					<source>1.6</source>
					<target>1.6</target>
				</configuration>
			</plugin>
			<!-- Source -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-source-plugin</artifactId>
				<version>2.4</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>jar-no-fork</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<!-- Javadoc -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-javadoc-plugin</artifactId>
				<version>2.10.3</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>jar</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-gpg-plugin</artifactId>
				<version>1.5</version>
				<executions>
					<execution>
						<id>sign-artifacts</id>
						<phase>verify</phase>
						<goals>
							<goal>sign</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<skip>false</skip>
				</configuration>
			</plugin>
		</plugins>
	</build>
	<distributionManagement>
		<snapshotRepository>
			<id>oss</id>
			<url>https://oss.sonatype.org/content/repositories/snapshots/</url>
		</snapshotRepository>
		<repository>
			<id>oss</id>
			<url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
		</repository>
	</distributionManagement>
</project>
```

