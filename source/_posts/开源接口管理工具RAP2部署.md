---
title: 开源接口管理工具RAP2部署
date: 2019-01-02 10:02:40
tags: 问题记录
categories: 开源工具
---
# 前言
此文是我在服务器部署RAP2过程中的流程及问题记录。
* 服务器版本：centos 6.9
* 数据库版本：MySQL 5.7
* Node版本：8.9.4
* redis版本：4.0

# 大纲
整个过程分三块：
* MySQL数据库安装
* 后端数据API服务器rap2-delos部署
* 前端静态资源rap2-dolores部署
> redis已经安装好了，我就没装，请自行搜索相关教程

# MySQL数据库安装
[参考](http://blog.51cto.com/phpervip/2063092)
> 原文step5、step8指令有误
* Step1: 检测系统是否自带安装mysql
```
yum list installed | grep mysql
```
* Step2: 删除系统自带的mysql及其依赖
```
yum -y remove mysql-libs.x86_64
```
* Step3: 给CentOS添加rpm源，并且选择对应的源。CeontOS6只能用el6的。
[官网地址](http://dev.mysql.com/downloads/repo/yum/)
```
下载源
wget dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
安装源
yum localinstall mysql-community-release-el6-5.noarch.rpm
查看启用情况
yum repolist all | grep mysql
设置启用mysql57
yum repolist enabled | grep "mysql.-community."
```
* Step4:安装mysql 服务器 命令：
 ```
 yum install mysql-community-server
```
* Step5:启动MySQL服务
```
service mysqld start 
```
* Step6:查看root 临时密码
```
grep 'temporary password' /var/log/mysqld.log
```
* Step7:修改root密码
```
mysql -uroot -p
出现密码输入框，使用刚才查看的临时密码登录
登陆之后输入以下命令修改密码其中MyNewPass2!为新密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass2!';
```
> 密码必须包含字母大小写、数字和特殊字符
* Step8:设置远程用户
```
create user 'root'@'%' identified by 'MyNewPass2!';
grant all privileges on *.* to 'root'@'%';
flush privileges;
```
> 新版本MySql把创建账户和赋予权限分开了，所以必须要两条命令
## 问题解决
Q: navicat for mysql 链接时报错：1251-Client does not support authentication protocol requested by server
A: 参考[此文](https://my.oschina.net/u/3295928/blog/1811804)，使用如下命令解决：
```
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'MyNewPass2!';
```

# 后端数据API服务器rap2-delos部署
* 1、输入以下指令，并用新密码登录
```
mysql -uroot -p
```
* 2、创建数据库
```
CREATE DATABASE IF NOT EXISTS RAP2_DELOS_APP DEFAULT CHARSET utf8 COLLATE utf8_general_ci
```
* 3、初始化
```
npm install
```
* 4、修改生产环境配置src/config/config.prod.ts
包含数据库密码的修改、redis服务器地址、mySql数据库地址的指定，**localhost**都要换成具体ip
> 由于step6初始化数据库表时默认针对的是开发环境，而我是跑的生产环境，所以顺便做了以下修改：
> * package.json中修改create-db的NODE_ENV为production
> * config.prod.ts中的database改成RAP2_DELOS_APP
* 5、安装 && TypeScript编译
```
npm install -g typescript
npm run build
```
* 6、初始化数据库表
```
npm run create-db
```
* 7、启动生产服务器
```
npm start
```

# 前端静态资源rap2-dolores部署
* 1、初始化
```
npm install
```
* 2、修改生产环境配置src/config/config.prod.js
配置前面启动的rap2-delos服务器的地址
* 3、编译react生产包
```
npm run build
```
* 4、用serve命令或nginx服务器路由到编译产出的build文件夹作为静态服务器
```
serve -s ./build -p 80
```
> 我用的是pm2
```
pm2 serve ./build 8082 --name rap2-dolores
```
