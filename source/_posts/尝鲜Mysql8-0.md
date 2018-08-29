---
title: 尝鲜Mysql8.0
date: 2018-05-04 11:17:21
tags: [Mysql,linux]
categories:
    - linux
---

> MySQL 8 正式版 8.0.11 已发布，官方表示 MySQL 8 要比 MySQL 5.7 快 2 倍，支持json

> 以下为centos 7.4下 编译安装Mysql8.0 的记录，大约需要1小时

<!-- more -->
### 安装前

#### 环境
```bash
# 版本
$ cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core)

# 环境依赖
$ yum -y install wget  cmake gcc gcc-c++ ncurses  ncurses-devel  libaio-devel  openssl openssl-devel
```

#### 源码
```bash
$ wget https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-boost-8.0.11.tar.gz
```

```bash
# 解压
$ tar -zxvf mysql-boost-8.0.11.tar.gz
$ cd mysql-8.0.11/
$ pwd
/root/download/mysql-8.0.11/
```

### 安装
#### 
```bash
#安装目录 
$ mkdir -p /usr/local/mysql   

#数据目录
$ mkdir -p /usr/local/mysql/data

# 编译前配置
# 最后一个目录为源码存放目录
$ cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DSYSCONFDIR=/etc -DMYSQL_TCP_PORT=3306 -DWITH_BOOST=/root/download/mysql-8.0.11/boost

$ make
# 会等很长一段时间 喝杯咖啡吧

$ make install
```

### 配置
