---
title: 《深入浅出Node.js》学习笔记
date: 2019-01-15 10:02:50
tags: 学习
categories: 服务端
---
# node介绍
## 特点
* 异步I/O
* 事件与回调函数
* 单线程

## 单线程弱点
* 无法利用多核CPU
* 错误会引起整个应用退出，应用的健壮性值得考验
* 大量计算占用CPU导致无法继续调用异步I/O

## 解决方案
* Nnigx 反向代理，负载均衡，开多个进程，绑定多个端口
* child_process fork出多个进程分担运算任务，共同监听一个端口
* cluster模块，对child_process的一层封装，同时自带负载均衡，默认使用Round-Robin算法实现，master监听TCP端口，并根据RR算法将请求交给钦定的worker进行处理。
* pm2进程管理工具，基于cluster模块实现负载均衡
> 多进程监听同一端口，存在“惊群”现象，效率低下，而且由于处理的进程是哪个不可控，因此没办法做负载均衡。
nginx做负载均衡，把请求反向代理给node，缺点是与nginx耦合度太高。

## 应用场景
* I/O密集型，明显优势
* CPU密集型，得益于V8的深度优化，node的运算能力并不差，只是受限于单线程带来的影响，可通过C/C++扩展进一步压榨单个CPU的性能或者通过子进程的方式利用多核CPU

# 模块机制
node引入模块经历了3个步骤
* 路径分析
* 文件定位
* 编译执行

## 路径分析
* **核心模块**，如http、fs、path等
* . 或 .. 开始的相对路径**文件模块**
* 以 / 开始的绝对路径**文件模块**
* 非路径形式的**文件模块**，如自定义的connect模块

## 文件定位
* 扩展名分析，按照js、json、node的次序
* 目录分析和包
> 为提升性能，引入json、node文件时，带上扩展名会比不带扩展名更快

## 编译执行
* js文件。通过fs模块同步读取文件后编译执行
* node文件(C/C++编写的扩展文件)。通过dlopen()方法加载最后编译生成的文件
* json文件。通过fs模块同步读取文件后，用JSON.parse()解析返回结果
* 其余扩展名文件。当做js文件载入
> 在编译的过程中，Node对获取的JavaScript文件内容进行了头尾包装。在头部添加
了(function (exports, require, module, __filename, __dirname) {\n，在尾部添加了\n});
以此达到作用域隔离的目的。

## 模块调用栈
* C/C++内建模块，属于核心模块，提供API给JavaScript核心模块和第三方JavaScript文件模块调用
* JavaScript核心模块，两个职责：一种是C/C++内建模块的封装层和桥接层，供文件模块调用；一种是纯粹的功能模块，不和底层打交道。
* 文件模块，通常由第三方编写。

# 异步I/O
## 实现方式
* read：重复检查I/O状态
* select/poll：通过数组/链表储存事件状态，达到一次查询多个I/O完成状态的目的
* epoll/kqueue：进入轮询时未检测到I/O完成就进入休眠(休眠时CPU几乎闲置)，直到事件通知、执行回调(kqueue的实现方式与epoll类似，不过它仅在FreeBSD系统下存在。)
* 线程池模拟异步I/O
> Node单线程指JavaScript执行在单线程，内部I/O是有线程池的

## Node的异步I/O实现
![](/images/article/《深入浅出Node.js》学习笔记/Node的异步IO实现.png) 
> nginx也摒弃了多线程的方式，采用了事件驱动来处理并发。node与nginx的比较
> * nginx采用C编写，性能更高，更适合做web服务器，可用于反向代理、负载均衡，并且处理静态资源的能力更佳
> * node场景更大，自身性能也不错，可以处理具体业务，比nginx更全面

缺少适用项目实践，决定暂缓继续学习。
有一个疑问：对于node做接口合并，到底是应该把接口数据一股脑丢给前端合适，还是把数据的组装也放在node合适？
to be continued...