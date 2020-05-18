---
title: Ubuntu安装Shadowsocks
date: 2016-02-02 15:29:02
tags: linux
toc : true
---

## ubunts版本
```bash
▶ cat /etc/issue
Ubuntu 14.04.1 LTS \n \l
```
<!-- more -->
## 安装步骤
* 安装pip
```bash
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

* 用pip安装 shadowsocks
```bash
pip install shadowsocks

```

* 创建配置文件
```bash
cd ~
vi shadowsocks.json
```
* 编写内容
```js

{
    "server":"0.0.0.0",
    "server_port":8388, // 配置访问的端口
    "local_port":1080,
    "password":"you_password", // 配置你自己的密码
    "timeout":600,
    "method":"aes-256-cfb"
}

```

## 以服务方式运行Shadowsocks
```bash
# 启动方法
▶ ssserver -c shadowsocks.json -d start
INFO: loading config from shadowsocks.json
2016-02-02 03:24:38 INFO     loading libcrypto from libcrypto.so.1.0.0
started

#停止方法
ssserver -c shadowsocks.json -d stop

```

## 客户端方法和教程
客户端配置好`服务器地址`,`端口`和`密码`和`加密方式`就能连接了
客户端工具下载[http://www.ishadowsocks.net/](http://www.ishadowsocks.net/)


