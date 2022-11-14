---
title       : "理解iptables的工作流程"
author      : "mingo"
date        : "2019-09-22 11:28:53"

layout      : post
category    : unix
tag         : ["net","原创"]
blog        : true
star        : false
---

## 概述

作为一个后端开发同学, 我们经常与`iptables`打交道, 熟练掌握`iptables`的工作流程及规则配置是我们进阶路上必备的技能

在理解了`iptables`的表的规划, 规则引擎, 工作流程设计之后, 对于以后我们的软件开发也是很有借鉴作用的

本篇文章尽量由浅入深的介绍`iptables`的原理, 用法, 以及一些最佳实践, 方便大家尽快的对`iptables`有个全面清晰的认识, 进而可以放心大胆的玩耍

## `iptables`是什么

对于初接触`iptables`的新手来说, 在配置规则之前要学习的内容很多, 表, chains, 规则...

命令参数又多又长, 感觉很复杂, 无从下手; 配置完之后, 又不知如何测试规则有无生效

`iptables` 是一个配置`Linux`内核防火墙的命令行工具，是`netfilter`项目的一部分。术语`iptables`也经常代指该内核级防火墙。
`iptables`可以直接配置，也可以通过许多前端和图形界面配置。

`iptables`用于ipv4，`ip6tables`用于ipv6。

既然是防火墙, 对于防火墙我们有一些基本的诉求

- 拦住指定特征的请求
- 放过指定特征的请求
- 可以对某些请求进行转发

### `iptables`工作流程

先来个完整的图来感觉下

<img src="/assets/images/linux/tables_traverse.jpg" width="480px" height="800px" alt="iptables完整流程图"/>

对于`centos7`系统, 可以使用`systemctl status iptables` 来查看当前各个表的配置; `centos6`使用`service iptables status`命令查看

在大多数使用情况下都不会用到 raw，mangle 和 security 表。下图简要描述了网络数据包通过 iptables 的过程：

```shell
                              XXXXXXXXXXXXXXXXXX
                              X     Network    X
                              XXXXXXXXXXXXXXXXXX
                                       +
                                       |
                                       v
 +-------------+              +------------------+
 |table: filter| <---+        | table: nat       |
 |chain: INPUT |     |        | chain: PREROUTING|
 +-----+-------+     |        +--------+---------+
       |             |                 |
       v             |                 v
 [local process]     |           ****************          +--------------+
       |             +---------+ Routing decision +------> |table: filter |
       v                         ****************          |chain: FORWARD|
****************                                           +------+-------+
Routing decision                                                  |
****************                                                  |
       |                                                          |
       v                        ****************                  |
+-------------+       +------>  Routing decision  <---------------+
|table: nat   |       |         ****************
|chain: OUTPUT|       |                +
+-----+-------+       |                |
      |               |                v
      v               |      +-------------------+
+--------------+      |      | table: nat        |
|table: filter | +----+      | chain: POSTROUTING|
|chain: OUTPUT |             +--------+----------+
+--------------+                       |
                                       v
                               XXXXXXXXXXXXXXXXXX
                               X    Network     X
                               XXXXXXXXXXXXXXXXXX
```

我们按照`Chains`的视角来看整个工作流程, 大体有2种流程

- start -> PREROUTING -> INPUT -> OUTPUT -> POSTROUTING -> end
- start -> PREROUTING -> FORWARD -> POSTROUTING -> end

所以按着这个角度来画出流程就是下图: 

![按生命周期来看的流程图](/assets/images/linux/tables_traverse_flow.png)

`PREROUTING`, `POSTROUTING`等等这些在`iptables`里名叫`Chains`, 中文叫`链`; 因为每个`chain`下面我们可以定义一个个`规则`, 然后这些规则按定义的顺序去执行, 整个过程就像链条一样, 故因此得名(这个名字的由来是我瞎编的)

> 这个设计思路很多有路由相关的都是类似思路, 比较`Java Servlet`里的`Filter`, `Python django`里的`middleware`都是采用类似的思想实现; 

> 使用链条的结构好处就是, 我们可以把链条里每个环抽象成一个组件, 这样使用者就可以对环进行任意组合,编排,插拔; 功能强大, 灵活性高, 实现又容易, 架构也稳定

下面逐一解释下这5条链条分别做的事情

### PREROUTING

- 具体就是在`INPUT`与`FORWARD`链之前统一执行的规则

### INPUT

- 包到目标机器的规则执行; 比如白名单, 只允许指定主机访问本机或者内网机器访问

### FORWARD

- 包需要转发的转发规则

### OUTPUT

- 目标机器到包的规则执行; 比如屏蔽某个网站的防问

### POSTROUTING

- 具体就是在`OUTPUT`与`FORWARD`链之后统一执行的规则

> 这5条链条的作用是我自己使用经验总结, 不一定正确, 后续有新体悟再来补充

`iptables` 定义了5张表, 用来各司其职

- raw: 用于配置数据包，raw 中的数据包不会被系统跟踪。网址过滤
    + 包含: `PREROUTING`, `OUTPUT`

- mangle: 用于对特定数据包的修改, 用于实现服务质量
    + 包含: `PREROUTING`, `INPUT`, `FORWARD`, `OUTPUT`, `POSTROUTING`

- nat: 用于网络地址转换（例如：端口转发), 网关路由器
    + 包含: `PREROUTING`, `INPUT`, `OUTPUT`, `POSTROUTING`

- filter: 是用于存放所有与防火墙相关操作的默认表。防火墙规则
    + 包含: `INPUT`, `FORWARD`, `OUTPUT`
    + 我们最常用的表

- security: 用于`强制访问控制`网络规则
    
## 怎么使用

我们使用`iptables`就是在具体的表下的相应`Chains`下添加/编辑规则, `iptables`就把这些规则组成规则引擎

当本机收到一个请求时, 先经过`PREROUTING`链的所有规则过滤, 再经过`INPUT`链的所有规则过滤才达到业务方; 
然后如果业务方需要给对方回包, 回包依次先经过`OUTPUT`链, `POSTROUTING`链的所有规则;
如果在某一`Chains`匹配某一个规则, 就执行相应的规则的目标; 

当然了, 上面的描述是比较简单化的

### 规则匹配条件

- `-s`: 源地址
- `-d`: 目标地址
- `--sport`: 源端口
- `--dport`: 目标端口

### 规则匹配原则

- 如果规则匹配上且目标是`DROP`, `REJECT`, 直接结束
- 如果满足当前规则且目标是`ACCEPT`, 则继续下一个规则
- 重复上面2条则, 直到链条结束

> 规则的次序非常关键，谁的规则越严格，应该放的越靠前，而检查规则的时候，是按照从上往下的方式进行检查的。

### 1. 关闭`iptables` (不推荐)

> iptables -F   # Flush the selected chain

> iptables -X   # Delete the selected chain

### 2. 查看`iptables`

> iptables -nvL

> iptables -nvL --line-numbers  # 在删除规则时很好用

### 3. 添加一条规则

> iptables -I INPUT -p tcp -s 10.0.0.85 --dport 17500 -j ACCEPT -m comment --comment "Friendly Dropbox"

- `-I`: 表示在链条前面插入规则, 如果想在后面添加规则用`-A`; 参数可选值就是上面的5条链条
- `-p`: 用来指定通信协议, 可选值有: `all`, `tcp`, `udp`, `udplite`, `icmp`, `esp`, `ah`, `sctp`
- `-s`: 包来源地址
- `--dport`: 表示目标端口, 一般就是本机了; 源端口是`--sport`
- `-j`: 专业名词叫目标, 常见的有`ACCEPT`, `DROP`, `REJECT`, `SNAT`, `DNAT`, `REDIRECT`, `LOG`
- `-m`: 执行扩展模块, 比如`comment`, 对规则添加评论注释; `state`模块, 记录网络包状态, 记录了状态就能达到"连接追踪"的功能

> iptables -I INPUT -p all -j ACCEPT -m state --state RELATED,ESTABLISHED

> iptables -A INPUT -p all -j REJECT --reject-with icmp-host-prohibited

上面的这2条规则配合使用能达到只有本机发出的包接受, 其它主机主动发过来的包都会被拒绝

### 4. 如果禁止主机访问某个地址

> iptables -A OUTPUT -d 1.2.3.4 -j REJECT

### 5. 更新规则

> iptables -R INPUT 1 -p tcp -j DROP

### 6. 删除规则

> iptables -D INPUT 1   # 规则编号可以通过 `iptables -L --line-numbers` 查询

## 进阶

### 1. iptables备份与恢复

有些不熟悉iptables的人经常会执行`iptables -F`来清空iptables的配置, 这样会让我们的服务器处于很危险的状态中

所以经常备份iptables配置是个好习惯

- 备份:

> iptables-save > /etc/sysconfig/iptables

- 恢复

> iptables-restore < /etc/sysconfig/iptables

## 参考
- [iptables,这篇文章里列举了好多实用的命令](https://wangchujiang.com/linux-command/c/iptables.html)
- [iptables概念,作者对iptables有自己的独到理解](https://www.zsythink.net/archives/1199)
- [wiki-Iptables,里面引用的资料比较多比较全面](https://wiki.archlinux.org/index.php/Iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [wiki-NAT](https://en.wikipedia.org/wiki/Network_address_translation)
