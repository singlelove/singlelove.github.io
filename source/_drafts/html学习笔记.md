---
title: html学习笔记
tags: 学习
categories: html
---
# script加载顺序
[参考](http://www.html5rocks.com/en/tutorials/speed/script-loading/)

## 普通情况
规范：同时下载，顺序执行
浏览器：同上

## defer属性
规范：同时下载，在DOMContentLoaded事件完成后顺序执行，忽略没有"src"属性的标签
IE&lt;10：可能在执行上一个js的中途中断，去执行下一个js
opera：不支持defer
其他浏览器：可能不会忽略没有"src"属性的标签

## async属性
规范：同时下载，下载完成就执行
IE<=10, Opera：不支持async

## 动态加载，并且async设置为false时
规范：同时下载，下载完成后尽快顺序执行
Firefox&lt;3.6, Opera：不支持async，但是会按顺序执行(按照标签的添加顺序)
Safari 5：支持async，但是这种情况下，跟async的情况一样，下载完成就执行，没有顺序
IE<10：不支持async，但是可以通过"onreadystatechange"实现，参考上面的网站
IE<=10, Opera：不支持async，下载完成就执行，没有顺序
其他浏览器：按照规范执行

