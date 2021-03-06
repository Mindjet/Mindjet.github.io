---
layout: post
title: TCP/IP 网络层
date: 2018-03-20 16:32:05 +0800
categories: [coding, network, tcp/ip]
permalink: /:categories/:title
index: 15
---

> 网络层主要是定义网络地址和路由寻径，IP 是网络层的实现。


## IP 地址

TCP/IP 协议网络上每一个网络适配器都有一个 IP 地址。该地址由 32 bit 来定义，通常我们分为 8 bit*4 的格式分为 4 段，每一段又以十进制的格式来显示，如：192.168.0.1。

### IP 地址分类

IP 地址分为两部分：**网络 ID（network id）**和**主机 ID （host ID）**。

但是具体 32 bit 中哪部分表示网络 ID，哪部分表示主机 ID 并没有强制规定。有些网络中主机数较多，那么主机 ID 所占的位数就多，反之亦然。

一般地，IP 地址分为分为以下几类：

- A 类：IP 地址前 8 bit 表示网络 ID，后 24 bit 表示主机 ID，且二进制 IP 地址以 **0** 开头；
- B 类：IP 地址前 16 bit 表示网络 ID，后 16 bit 表示主机 ID，且二进制 IP 地址以 **10** 开头；
- C 类：IP 地址前 24 bit 表示网络 ID，后 8 bit 表示主机 ID，且二进制 IP 地址以 **110** 开头；

![](https://user-gold-cdn.xitu.io/2017/4/4/e8cb176a9fc070cde9b963c4869282f4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

注意：

- IP 地址第一段大于 223 的属于 D 类和 E 类地址，比较特殊，不做介绍；
- 若主机 ID 全为 0，则该 IP 代表网络本身，如 192.10.0.1 代表 192.10.0 这一个 C 类地址网络；
- 若主机 ID 全为 1，代表着广播地址，用于向该网络中所有主机发送消息的，如 192.10.0.255 是网络 ID 为 192.10.0 网络的广播地址。
- A 类地址中，整个 127 网络都是环回（loopback）地址，用以测试 TCP/IP 是否正常


*PS: IP 地址由美国国防数据网 DDN 的网络信息中心 NIC 进行分配。*

## 子网

### 子网 ID

比如一个 A 类网络，下面可以有 2^24 > 1600w 台主机，控制起来十分灵活，我们想要将该网络切分为更小的网络来调控，子网就是为了解决这种问题的。

我们上面提到，我们将一个 IP 地址分为网络 ID 和 主机 ID 两部分，如果需要使用子网，则需要**子网 ID（subnet id）**，它会占用主机 ID 的空间。

简单地举个例子，比如学校分配到了一个 C 类网络 192.10.10.0，其网络 ID 即为 192.10.10，机主 ID 为 8 bit，共可容纳 2^8 - 2 = 254 台主机。现在我们将 IP 地址分为两个子网，一个给教学区用，一个给办公区用。有人说了，你直接再搞一个 C 类网络不就好了比如 192.10.11.0？这确实方便，但是这将造成 IP 地址的极大浪费。

### 子网掩码

我们可以利用子网 IP，将学校网络切分为 2 个子网。这里为了方便下面说明，先介绍一下**子网掩码（subnet mask）**的概念。

子网掩码跟 IP 地址一样，也是 32 bit 来表示的，它含有连续的 1 和 0，而且一定是 1 排在前面。其中 1 表示的是已经切分出子网的网络 ID，0 表示的是主机  ID。

这么说有点抽象，我们举个例子吧。比如一个 C 类网络，193.10.10.0，我们可以将其想象成整个大网络其中的一个子网，它的网络 ID 为 24 bit，那么其子网掩码就为 255.255.255.0，表示为 193.10.10.0/24。

### 子网切分

现在，我们现在将学校网络切分为两个子网，子网 ID 需要占用 2 bit（至于为什么不是 1 bit 下一段会讲） 的主机 ID 位，此时可以将当前网络切分为 2 个子网（为什么是 2 个子网而不是 4 个子网，下面也会说到）。

那么此时切分出来的两位子网的信息如下：

|        网络        |     网络地址      |     广播地址      |
| :--------------: | :-----------: | :-----------: |
| 192.10.10.128/26 | 192.10.10.128 | 192.10.10.191 |
| 192.10.10.64/26  | 192.10.10.64  | 192.10.10.127 |

需要对上面的结果解释一下。

首先，为什么 2 bit 的子网 ID 只能切分出 2 个子网，不应该有 00,01,10,11 四种情况四个子网吗？确实是，但是一般全 0 和全 1 的子网 ID 是不可用的，所以只剩下 01,10 两种情况了（具体可见[这篇文章](http://blog.csdn.net/acgjun/article/details/28276619)）。

既然只有 01,10 两种情况，那么此时将 192.10.10.0 这个网络切分为：192.10.10.**01**000000 和 192.10.10.**10**000000 这两个子网，为了看得更清晰我把最后 8 bit 写成二进制并且把子网 ID 加粗显示。

对于这两个子网，其主机 ID 位数就剩下 6 bit 了，将主机 ID 为全置 0 则对应子网的网络地址，全置 1 则对应子网的广播地址。

## ARP 协议

IP地址只是主机在网络层中的地址，若要将网络层中传送的数据包交给目的主机，必须知道该主机的 MAC 地址。ARP，地址解析协议，就是将 IP 地址映射为 MAC 地址；反之有个 RARP 协议，是将MAC 地址映射为 IP 地址。

*link: [MAC地址](https://mindjet.github.io/coding/tcp-ip-link-layer)*

## IP 路由

说了这么多，那么在网络层上，主机之间是如何通信的？

![](https://user-gold-cdn.xitu.io/2018/3/21/16248bcaab4a8999?w=816&h=605&f=png&s=22168)

考虑两种情况：网段内通信和不同网段（网络）间通信。

### 网段内通信

#### 获取目标 MAC 地址

比如上图的主机 A 和 服务器 A 都处于 192.168.0.0 网络中，主机 A 若想要发送数据给服务器 A，此时主机 A 知道自己的 IP 地址和 MAC 地址，同时还知道服务器 A 的 IP 地址，但是不知道它的 MAC 地址。

这时，主机 A 通过子网掩码计算出源地址（主机 A 的 IP 地址）和目标地址（服务器 A 的 IP 地址）是处于同个网段内，那么此时主机 A 就向本网段发送一个 ARP 请求，该请求包中所带的关键信息如下：

```
发送方IP地址：192.168.0.2		|		发送方MAC地址：XX-XX-XX-XX-XX-XX
接收方IP地址：192.168.0.3		|		接收方MAC地址：FF-FF-FF-FF-FF-FF
```

包中除 接收方 MAC 地址外，所有信息都是未知的，接收方 MAC 地址设为广播 MAC 地址。

这个 ARP 包在整个网段内都可以接收到，服务器 A 自然也能接收到。服务器 A 接收到后发现接收方 IP 是自己的 IP，那么就将自己的 MAC 地址填上去并返回完整的 ARP 回应包：

```
发送方IP地址：192.168.0.2		|		发送方MAC地址：XX-XX-XX-XX-XX-XX
接收方IP地址：192.168.0.3		|		接收方MAC地址：YY-YY-YY-YY-YY-YY
```

现在，主机 A 就获取到源 IP，源 MAC，目标 IP 和目标 MAC 了。此时再将这些信息和一开始的数据包封装在一起就可以发送出去了。

#### ARP 缓存

实际上，每台主机都有维护一份 ARP 缓存表，记录着与自己发生过通信的所有的直接相邻设备或主机的 MAC 地址，但ARP缓存表经过一段时间会自动删除。

这个过程其实就有点像写信，比如说李雷想向班里的一个人写信。

\- 李雷的名字（IP）和地址（MAC）自己肯定是知道的，家就住北京市奶子房吧。

\- 你同学的名字（IP）叫韩梅梅他肯定也知道，但是不知道她家（MAC）住哪儿。

\- 这时李雷就在班里的 QQ 群喊了一句：“韩梅梅，我叫李雷，家住北京奶子房，你家地址是什么？”（ARP 包）

\- 这时候班里所有人肯定都收到这条消息，但是他们想：“不又是叫我名字，不鸟他。”只有韩梅梅回应：“家住北京五道口的李雷，我是韩梅梅，我家住在北京南站东广场西出口~”（ARP 回应包）

\- 然后班里所有人肯定又都收到这条消息，但是他们想：“不又是叫我名字，不鸟他。”只有李雷默默地记下了她的地址。

\- 然后李雷就能成功把寄信人、寄信地址、收件人和收件地址给写全了，然后把包裹在信封里面的信寄出去了。

\- 下次李雷想写信给韩梅梅的时候，就不用再次去询问韩梅梅的地址了，因为他还记得上次写信时的地址（ARP缓存表），但是如果太久没写信，李雷就健忘了，还是得问韩梅梅家住哪儿。

\- md，李雷真矫情，都用 QQ 了还写信，去 QQ 爱啊淦。

### 不同网段通信

不同网段间设备的通信就要依赖路由器了。

比如说主机 A 想要发送数据给主机 B，用子网掩码算了一下，两者并不属于同个网络。也就是说主机 A 不知道要发给谁了，这种情况会默认发送到网关，发送到网关之前当然也需要知道网关的 MAC 地址，网关的 IP 是知道的（192.168.0.1），网关 MAC 的获取方法就跟上面的例子一样。

得到网关的 MAC 地址后，我们就可以将数据包发送出去了，但是注意，此时的目标 IP 是主机 B 的 IP，但是目标 MAC 是网关 MAC。

```
源IP：主机A的IP | 源MAC：主机A的MAC | 目标IP：主机B的IP | 目标MAC：路由器X接口MAC | 数据
```

图中我们假设网关本身是路由器，如果网关是一台普通的 PC，那么需要将数据发给路由器处理。路由器接收到数据包后发现，Y 端口和目标 IP 处于同个网段，这时通过 ARP 来确定目标 IP 的 MAC 地址，重新修改数据包中目标 MAC 地址和源 MAC 地址即可。（注意源 MAC 和目标 MAC 在不同网段的通信过程中是会一直改变的）

```
源IP：主机A的IP | 源MAC：路由器Y接口MAC | 目标IP：主机B的IP | 目标MAC：主机B的MAC | 数据
```

当然，这种网络接口是很简单的，只需要一个路由就可以实现两个不同网络间的通信（这种路由称直接路由），而实际情况下，数据包可能需要经过很多个路由才能达到目标主机。



