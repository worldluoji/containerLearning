# iptables

## 1. 什么是iptables?
实际上，iptables 只是一个操作 Linux 内核 Netfilter 子系统的“界面”。

顾名思义，Netfilter 子系统的作用，就是 Linux 内核里挡在“网卡”和“用户态进程”之间的一道“防火墙”。

iptables 是集成在 Linux 内核中的包过滤防火墙系统。使用 iptables 可以添加、删除具体的过滤规则，iptables 默认维护着 4 个表和 5 个链，所有的防火墙策略规则都被分别写入这些表与链中。

“四表”是指 iptables 的功能，默认的 iptables 规则表有 filter 表（过滤规则表）、
nat 表（地址转换规则表）、mangle（修改数据标记位规则表）、raw（跟踪数据表规则表）：

- 1) filter 表：控制数据包是否允许进出及转发，可以控制的链路有 INPUT、FORWARD 和 OUTPUT。
- 2) nat 表：控制数据包中地址转换，可以控制的链路有 PREROUTING、INPUT、OUTPUT 和 POSTROUTING。
- 3) mangle：修改数据包中的原数据，可以控制的链路有 PREROUTING、INPUT、OUTPUT、FORWARD 和 POSTROUTING。
- 4) raw：控制 nat 表中连接追踪机制的启用状况，可以控制的链路有 PREROUTING、OUTPUT。

“五链”是指内核中控制网络的 NetFilter 定义的 5 个规则链。
每个规则表中包含多个数据链：INPUT（入站数据过滤）、OUTPUT（出站数据过滤）、FORWARD（转发数据过滤）、
PREROUTING（路由前过滤）和POSTROUTING（路由后过滤），防火墙规则需要写入到这些具体的数据链中。

<img src="iptables中的表和链.webp" width="70%"/>

五张表的处理是有顺序的。当数据包到达某一条链时，会按照 RAW、MANGLE、NAT、FILTER、SECURITY 的顺序进行处理。

2. Chain
IP 包“一进一出”的两条路径上，有几个关键的“检查点”，它们正是 Netfilter 设置“防火墙”的地方。
在 iptables 中，这些“检查点”被称为：链（Chain）。

<img src="Chain.webp" width="70%" />

这些“检查点”对应的 iptables 规则，是按照定义顺序依次进行匹配的。

当一个 IP 包通过网卡进入主机之后，它就进入了 Netfilter 定义的流入路径（Input Path）里。

IP 包要经过路由表路由来决定下一步的去向。而在这次路由之前，Netfilter 设置了一个名叫 PREROUTING 的“检查点”。
在 Linux 内核的实现里，所谓“检查点”实际上就是内核网络协议栈代码里的 Hook（比如，在执行路由判断的代码之前，内核会先调用 PREROUTING 的 Hook）。

而在经过路由之后，IP 包的去向就分为了两种：
- 第一种，继续在本机处理；
- 第二种，被转发到其他目的地。

我们先说一下 IP 包的第一种去向。
- 1) IP 包将继续向上层协议栈流动。
在它进入传输层之前，Netfilter 会设置一个名叫 INPUT 的“检查点”。
到这里，IP 包流入路径（Input Path）结束。

- 2) 接下来，这个 IP 包通过传输层进入用户空间，交给用户进程处理。
而处理完成后，用户进程会通过本机发出返回的 IP 包。
这时候，这个 IP 包就进入了流出路径（Output Path）。

- 3) IP 包首先还是会经过主机的路由表进行路由。路由结束后，Netfilter 就会设置一个名叫 OUTPUT 的“检查点”。
然后，在 OUTPUT 之后，再设置一个名叫 POSTROUTING“检查点”


第二种去向
- 1) 这个 IP 包不会进入传输层，而是会继续在网络层流动，从而进入到转发路径（Forward Path）。
在转发路径中，Netfilter 会设置一个名叫 FORWARD 的“检查点”。
- 2) 在 FORWARD“检查点”完成后，IP 包就会来到流出路径。而转发的 IP 包由于目的地已经确定，
它就不会再经过路由，也自然不会经过 OUTPUT，而是会直接来到 POSTROUTING“检查点”。

POSTROUTING 的作用，其实就是上述两条路径，最终汇聚在一起的“最终检查点”。

在有网桥参与的情况下，上述 Netfilter 设置“检查点”的流程，实际上也会出现在链路层（二层），
并且会跟上面讲述的网络层（三层）的流程有交互。

这些链路层的“检查点”对应的操作界面叫作 ebtables.

<img src="iptables principle.webp" />

从上图中可以看到，还有其它的一些check和prerouting.


## 3. example

iptables -A FORWARD -d $podIP -m physdev --physdev-is-bridged -j KUBE-POD-SPECIFIC-FW-CHAIN
iptables -A FORWARD -d $podIP -j KUBE-POD-SPECIFIC-FW-CHAIN

第一条 FORWARD 链“拦截”的是一种特殊情况：它对应的是同一台宿主机上容器之间经过 CNI 网桥进行通信的流入数据包。
--physdev-is-bridged的意思就是，这个FORWARD链匹配的是，通过本机上的网桥设备，发往目的地址是podIP的IP包。
当然，如果是像 Calico 这样的非网桥模式的 CNI 插件，就不存在这个情况了。

第二条 FORWARD 链“拦截”的则是最普遍的情况，即：容器跨主通信。
这时候，流入容器的数据包都是经过路由转发（FORWARD 检查点）来的。

不难看到，这些规则最后都跳转（即：-j）到了名叫 KUBE-POD-SPECIFIC-FW-CHAIN 的规则上。
它正是网络插件为 NetworkPolicy 设置的第二组规则。
而这个 KUBE-POD-SPECIFIC-FW-CHAIN 的作用，就是做出“允许”或者“拒绝”的判断。

## 4. 常用命令
### 1) 查看规则
iptables -nvL

各参数的含义为：
-L 表示查看当前表的所有规则，默认查看的是 filter 表，如果要查看 nat 表，可以加上 -t nat 参数。
-n 表示不对 IP 地址进行反查，加上这个参数显示速度将会加快。
-v 表示输出详细信息，包含通过该规则的数据包数量、总字节数以及相应的网络接口。

```
iptables-save

iptables-save指令可以将内核中当前的iptables配置导出到标准输出。通过IO重定向功能来定向输出到文件。

iptables-save -t nat               //输出nat表的信息
```

### 2) 添加规则
添加规则有两个参数分别是 -A 和 -I。
其中 -A 是添加到规则的末尾；-I 可以插入到指定位置，没有指定位置的话默认插入到规则的首部。

```
iptables -A INPUT -s 192.168.1.5 -j DROP
```

### 3) 修改规则
在修改规则时需要使用-R参数
把添加在第 6 行规则的 DROP 修改为 ACCEPT

iptables -R INPUT 6 -s 194.168.1.5 -j ACCEPT

### 4) 删除规则
删除规则有两种方法，但都必须使用 -D 参数。 
删除添加的第 6 行的规则:
```
iptables -D INPUT 6 -s 194.168.1.5 -j ACCEPT
或
iptables -D INPUT 6
```
注意，有时需要删除的规则较长，删除时需要写一大串的代码，这样比较容易写错，
这时可以先使用 -line-number 找出该条规则的行号，再通过行号删除规则。

### 5) 其它例子
拒绝转发来自 192.168.1.10 主机的数据，允许转发来自 192.168.0.0/24 网段的数据
```
$ iptables -A FORWARD -s 192.168.1.11 -j REJECT
$ iptables -A FORWARD -s 192.168.0.0/24 -j ACCEPT
```

丢弃从外网接口（eth1）进入防火墙本机的源地址为私网地址的数据包：
```
$ iptables -A INPUT -i eth1 -s 192.168.0.0/16 -j DROP
$ iptables -A INPUT -i eth1 -s 172.16.0.0/12 -j DROP
$ iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP
```

只允许管理员从 202.13.0.0/16 网段使用 SSH 远程登录防火墙主机
```
$ iptables -A INPUT -p tcp --dport 22 -s 202.13.0.0/16 -j ACCEPT
$ iptables -A INPUT -p tcp --dport 22 -j DROP
```

## 参考资料
- https://time.geekbang.org/column/article/413279