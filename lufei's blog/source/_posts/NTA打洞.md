---
title: NTA打洞
date: 2018-9-11 16:30:21
tags: 网络
categories: 网络
copyright: false
---

# NTA打洞

## 只有一方处于NAT设备后
此种情况是所有P2P场景中最简单的，它使用一种被称为“反向链接技术”来解决这个问题。大致的原理如下所述。

如图所示，客户端A位于`NAT`之后，它通过`TCP`端口1234连接到服务器的`TCP`端口1235上，NAT设备为这个连接重新分配了`TCP`端口62000。客户端B也通过TCP端口1234连接到服务器端口1235上。A和B从服务器处获知的对方的外网地址二元组`{IP地址:端口号}`分别为`{138.76.29.7:1234}`和`{155.99.25.11:62000}`，它们在各自的本地端口上进行侦听。

由于B 拥有外网`IP`地址，所以A要发起与B的通信，可以直接通过`TCP`连接到B。但如果B尝试通过TCP连接到A进行`P2P`通信，则会失败，原因是A位于`NAT`设备后，虽然B发出的`TCP` SYN请求能够到达NAT设备的端口62000，但`NAT`设备会拒绝这个连接请求。要想与`Client` A通信， B不是直接向A发起连接，而是通过服务器给A转发一个连接请求，反过来请求A连接到B（即进行反向链接），A在收到从服务器转发过来的请求以后，会主动向B发起一个`TCP`的连接请求，这样在`NAT`设备上就会建立起关于这个连接的相关表项，使A和B之间能够正常通信，从而建立起它们之间的`TCP`连接。

![反向链接示意图](/images/NAT打洞/反向链接.jpg)

## 双方都处于NAT后设备后
### NAT设备部署情况
```
两客户端都处于NAT设备背后也有很多情况,如:
都处在同一NAT设备后.
随直连的不是同一个NAT设备,中间有很多NAT转接,但顶端接入公网ip的NAT(网络服务商的路由)是同一个.
```

像第一种情况,就是在同一个以太网中,这种可以直接使用内网ip进行直连最优.
但是想第二种情况,因为嵌套在不同的内网中,都使用的保留ip地址,或许会存在与客户端本身所在的内网地址重复的可能性,所以还是以外网ip+端口来进行打洞最为稳妥.

因为有`Hairpin`技术,它可以让两台位于同一台NAT网关后面的主机,通过对方的公网端口互相访问.

### ICE
> ICE的全称Interactive Connectivity Establishment（互动式连接建立），由IETF的MMUSIC工作组开发出来的，它所提供的是一种框架，使各种NAT穿透技术可以实现统一。ICE跟STUN和TURN不一样，ICE不是一种协议，而是一个**框架**（Framework），它整合了STUN和TURN。

### TURN
TURN(Traversal Using Relays around NAT)TURN与STUN的共同点都是通过修改应用层中的私网地址达到NAT穿透的效果，异同点是TURN是通过两方通讯的“中间人”方式实现穿透

TURN服务器相比较STUN服务器,多了中继的功能,对于无法端对端的服务器,可以进包转发

### STUN
STUN服务器（Simple Traversal of User Datagram Protocol Through Network Address Translators）是一个轻量级协议,是基于UDP的完整的穿透NAT的解决方案.在进行NAT穿透时我们需要STUN服务器.

STUN服务器主要做了:
```
判断客户端的NAT类型(NAT分四种类型)。
接受客户端的请求,返回其需要连接的外网地址。
协调客户端间打洞。
```

### 具体实现

#### 是否处于NAT后(作为iOS肯定是在NAT后的,不用判断的)
客户端向STUN服务器发送UDP包,STUN将收到的包的IP包在UDP包中进行返回,客户端收到包后和自己的IP做比较,不一样则处在NAT后.
#### 判断NAT类型
当处在NAT后则要判断NAT类型,为了判断STUN服务器需要两个ip,ip1和ip2.

客户端当得知自己处在NAT设备后时,再向服务端ip1发送判断请求.

**FULL Cone NAT**
>服务器收到后从ip2向客户端的公网ip发送包,若客户端能收到,则为 完全雏形NAT.

**判断是否为 对称NAT**
>  客户端再从不同的port 想服务端发包,若客户端收到的外网端口(此处的端口号是指NAT表中ip后对应的端口号,而不是主机为外网服务的端口号)不一致,则为对称NAT,一致则不为对称NAT

**Restrict Cone NAT /Port Restrict NAT**
>服务端从ip1的不同端口向客户端发包,若客户端收到 则为`Restrict Cone NAT`,收不到则为 `Port RestrictNAT`.

#### 协调打洞
**Full Cone NAT**
>此时什么都不需要STUN做,只要知道对方公网ip+端口号 客户端间可以直接通讯

**Restrict Cone 或 Port Restrict**
>此时需要TURN服务器协调两客户端互相向对方公网ip发包, 因为在NAT设备看起来,互相发包都是在向公网发包,这时在NAT表上就会开启对指定外网ip+端口号的通道.这样就完成了打洞

**对称NAT**
>对于对称NAT是无法打洞的,因为客户端向STUN服务器发包的外网ip+端口号 与向其他客户端发包的外网ip+端口号是不一样的.所以无法沟通,这时就需要TURN服务器,两个客户端都对其进行长连接,进行包的转发.

## 参考链接
http://www.52im.net/thread-542-1-1.html
http://www.52im.net/thread-557-1-1.html
http://www.h3c.com.cn/MiniSite/Technology_Circle/Net_Reptile/The_Five/Home/Catalog/201206/747038_97665_0.htm

