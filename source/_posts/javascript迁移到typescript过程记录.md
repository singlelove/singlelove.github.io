---
title: javascript迁移到typescript过程记录
date: 2019-07-08 15:39:13
tags: 问题记录
---
# 前言
本文记录了项目由javascript迁移typescript过程中出现的问题。项目是JSP项目，页面使用React编写，使用了antd组件库，
通过webpack打包引入到页面中，构建过程使用了gulp，此为背景。

# 问题记录
## intellij的typescript版本落后
settings--languages--typescript中修改typescript version，改成custom目录

## TS7016: Could not find a declaration file for module 'react'
npm install @types/react --save

## TS1259: Module  can only be default-imported using the 'esModuleInterop'
add esModuleInterop into tsconfig.json

## tsc的报错包含了node_modules中的文件
add skipLibCheck into tsconfig.json
**待确认是否是最佳方案，会忽略所有声明文件**

## TS2339: Property 'propTypes' does not exist on type 'typeof xxx'.
改成放到class内部，定义static propTypes = {...}

## TS2339: Property 'props' does not exist on type 'ReactNode'.


## TS2339:Property 'xxx' does not exist on type 'Window'.


首先参考[官方文档](https://www.typescriptlang.org/docs/handbook/migrating-from-javascript.html)，

