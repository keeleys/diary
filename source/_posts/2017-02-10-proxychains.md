---
title: shadowsocks 科学上网的终端控制
date: 2017-02-10 14:32:43
tags: util
---


## shadowsocks 客户端

客户端推荐使用[shadowsocksR](https://github.com/shadowsocksr),配置很强大

## 终端
网上那些 set http proxy啥的都不好用。所以最后还是用这个自动代理工具了，


### 安装
```
brew install proxychains-ng
```
### 配置
修改配置 参考路径
/usr/local/Cellar/proxychains-ng/4.12/etc/proxychains.conf

这个端口跟你运行的shadowsocks配置的端口一致
```
[ProxyList]
socks5  127.0.0.1 1080
```

### 使用
然后
```
proxychains4 wget https://github.com/jeromelebel/MongoHub-Mac/archive/3.1.5b1.zip
```

当然可以设置下别名
```
alias pc="proxychains4"
```