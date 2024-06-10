---
date: 2024-06-10
tags:
  - "#课外知识"
  - 课外知识
title: plantegg 星球启示录 (1) (MySql抓包实验)
slug: 18:38
share: false
canonicalURL: 
keywords:
  - "#抓包"
description: 
series: 系列
lastmod: 
lang: cn
cover:
  image: ""
author: 
dir: posts
---


1. 通过Java和MySQL直连抓包测试, 新学到了一些知识点
 - 基于TShark可以直接在终端抓包并且显示, 这里可以方便的看到ip, port, 甚至sql语句  
 - 当最后出现连接断开时，看FIN ACK 这个包是从哪个端口发出来就知道是Client端的问题还是Server的问题  
 - 当Client端异常断开(client被kill, 请求socket timeout), Client会发送一个FIN ACK, 此时Server端并不会立刻回复, 而是等自己处理完, 尝试返回给Client, 然后Client会回一个RST  - 如果时Server端异常断开, 那么通过抓包可以直接看到Server端回了一个FIN ACK的包  
 - TCP请求一开始会把数据写入到OS, 然后Client端会从OS中读取, 如果出现请求返回后Client持续不读取数据, 就会导致TCP 0 Window, 以及导致 net_write_timeout(因为Client端的接收缓冲区满了, Server端也会发送失败, 最后把Server端的写入缓冲区也堆满, 然后Server端写入报错)
2. 基于tcp_rt去定位问题，能够快速拿到一个包的rt(server处理时间，网络传输时间)  
 - TCP-RT功能的配置说明  - 这里需要对linux的内核进行修改，也就是普通的环境是不带该功能的  
 - 考虑后续测试可以直接找个阿里的云机器去玩一下  
 - 可以通过cronjob，把tcp_rt日志定时(1min)输出到指定目录  
 - 通过awk等指令对日志进行解析，简化输出，例如server耗时，网络耗时请求QPS  - 记住公式 rt = 1 / (QPS /并发) 这里永远可以对上
3. 一些好玩的指令  
	1. 3.1 ssh -lntp | grep port 通过这个指令可以看到哪些user往监听的端口里发送请求, 可以用于各个服务之间的甩锅  
	2. 3.2 tshark 实际使用起来参数很多，蛮复杂的，日常中可以用起来，越用越熟练  
	3. 3.3 sysbench 性能测试工具, 查了下可以通过lua脚本来进行读写的性能测试, mongo也是默认支持的一些链接
4. 课外知识
- 解释为什么Client一直不处理数据会导致 [TCP Zero Window 就是要你懂TCP--性能和发送接收Buffer的关系](https://plantegg.github.io/2019/09/28/%E5%B0%B1%E6%98%AF%E8%A6%81%E4%BD%A0%E6%87%82TCP--%E6%80%A7%E8%83%BD%E5%92%8C%E5%8F%91%E9%80%81%E6%8E%A5%E6%94%B6Buffer%E7%9A%84%E5%85%B3%E7%B3%BB/)
- 基于tc 指令来让请求具有延迟方便复现问题[ Linux tc qdisc的使用案例 | plantegg](https://plantegg.github.io/2016/08/24/Linux%20tc%20qdisc%E7%9A%84%E4%BD%BF%E7%94%A8%E6%A1%88%E4%BE%8B/)
- 关于tshark具体的使用, 文章里是mysql, 切换成mongo的query应该也好搞 [WireShark之命令行版tshark](https://plantegg.github.io/2019/06/21/%E5%B0%B1%E6%98%AF%E8%A6%81%E4%BD%A0%E6%87%82%E6%8A%93%E5%8C%85--WireShark%E4%B9%8B%E5%91%BD%E4%BB%A4%E8%A1%8C%E7%89%88tshark/)
- 关于ss指令, 如何进阶的使用 [ss用法大全](https://plantegg.github.io/2016/10/12/ss%E7%94%A8%E6%B3%95%E5%A4%A7%E5%85%A8/)
