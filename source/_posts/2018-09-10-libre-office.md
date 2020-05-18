---
title: libre-office安装
date: 2018-09-10 15:19:56
tags:
---


## step1 (检查版本,目前是用redhat为例)

```
[root@test9 ~]# cat /proc/version
Linux version 3.10.0-327.36.2.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC) ) #1 SMP Mon Oct 10 23:08:37 UTC 2016

https://www.tecmint.com/install-libreoffice-on-rhel-centos-fedora-debian-ubuntu-linux-mint/
```

## step2(下载LibreOffice4.3版本)

```
wget https://downloadarchive.documentfoundation.org/libreoffice/old/4.3.7.2/rpm/x86_64/LibreOffice_4.3.7.2_Linux_x86-64_rpm.tar.gz
```

## step3(解压 安装LibreOffice)

```
$ tar -xzvf libreoffice*.tar.gz
$ cd libreoffice*/RPMS
$ sudo rpm -i *.rpm
```


## step4 (安装unoconv)

```
// 安装主体程序
yum install unoconv
// unoconv noarch 0.6-7.el7

// 安装语言包 (否则会乱码)
sudo yum install wqy-microhei-fonts wqy-zenhei-fonts
```

## step6 (开始使用咯..)

```
unoconv -f pdf -o test2.pdf 123.txt
```