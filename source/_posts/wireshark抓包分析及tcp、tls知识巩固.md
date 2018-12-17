---
title: wireshark抓包分析及tcp、tls知识巩固
date: 2018-12-14 11:24:44
tags: 学习
---
# 背景
某日，准备用fiddler抓包分析下某app的请求数据，结果发现无法捕获任何请求，google了一番，发现可能是app设置了不走系统代理
或者压根没用http请求数据，而是走了tcp。改用wireshark之后，成功捕获到了tcp，第一步障碍扫除。由于当初tcp掌握的不好，于是
先去恶补了一番。

# tcp巩固
## 三次握手
* C端向S端发送SYN(syn=j)标记，请求建立连接
> SYN：即是同步序列编号（Synchronize Sequence Numbers）
* S端收到后，向C端发送确认包，包含SYN(syn=k)和ACK(ack=j+1)标记
* C端收到确认包后，再向S端发送ACK(ack=k+1)标记，表示收到确认包
```
握手阶段：
序号   方向 seq   ack
1      A->B 10000 0
2      B->A 20000 10000+1=10001
3      A->B 10001 20000+1=20001
```
>seq可以理解为当前端当前时刻已发送的数据字节数，ack可以理解为当前端当前时刻已收到的数据字节数。参考
>[理解TCP序列号（Sequence Number）和确认号（Acknowledgment Number）](https://blog.csdn.net/a19881029/article/details/38091243)
>一文的解释。

...to be continued
参考资料：
* [TCP三次握手连接及seq和ack号的正确理解](https://blog.csdn.net/u013636377/article/details/50371933)
* [理解TCP序列号（Sequence Number）和确认号（Acknowledgment Number）](https://blog.csdn.net/a19881029/article/details/38091243)