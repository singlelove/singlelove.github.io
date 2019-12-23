---
title: web安全
date: 2018-06-15 14:55:23
tags: 学习
categories: 安全
---
# XSS攻击
可参考美团技术团队的[如何防止XSS攻击](https://tech.meituan.com/fe_security.html)一文
## 危害
* 窃取cookie
* 劫持流量实现恶意跳转

## 分类
* 反射型(非持久型)
* 储存型(持久型)
* DOM型

## 防范
* ~~输入过滤~~ (**不推荐，会引入很大的不确定性和乱码问题**)
* 标签过滤，script、img、a
* 编码，<>转码
* 限制输入长度

# CSRF跨站点请求伪造
可参考美团技术团队的[如何防止CSRF攻击](https://tech.meituan.com/fe_security_csrf.html)一文
## 原理
1、用户登录目标网站，并未退出
2、用户访问了攻击者的不安全网站，或者点击了攻击者的不安全链接
3、攻击者向目标网站发送请求，由于用户未登出，请求自动带上了cookie，被目标网站识别为用户的操作。

## 防范
* CSRF自动防御策略：同源检测（Origin 和 Referer 验证）。
* CSRF主动防御措施：Token验证 或者 双重Cookie验证 以及配合Samesite Cookie。
* 保证页面的幂等性，后端接口不要在GET页面中做用户操作。

## to be continued



