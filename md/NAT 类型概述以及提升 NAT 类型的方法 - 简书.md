> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/478a4acc9d74?from=singlemessage)

姓名：刘登贤

学号：14310116024

转载自：            http://www.right.com.cn/forum/thread-199299-1-1.html

                            http://blog.csdn.net/boywgw/article/details/48496151

                            https://baike.baidu.com/item/nat

                            http://www.cnblogs.com/my_life/articles/1908552.html

【嵌牛导读】：NAT 类型概述以及提升 NAT 类型的方法

【嵌牛鼻子】：关于 NAT 类型对网络的影响

【嵌牛提问】：NAT 类型有哪些？不同 NAT 类型有什么区别？提升 NAT 类型有什么好处？日常使用中怎样提高 NAT 类型？、

**一、什么是 NAT**

    NAT（Network Address Translation，网络地址转换）是 1994 年提出的。当在专用网内部的一些主机本来已经分配到了本地 IP 地址（即仅在本专用网内使用的专用地址），但现在又想和因特网上的主机通信（并不需要加密）时，可使用 NAT 方法。

这种方法需要在专用网连接到因特网的路由器上安装 NAT 软件。装有 NAT 软件的路由器叫做 NAT 路由器，它至少有一个有效的外部全球 IP 地址。这样，所有使用本地地址的主机在和外界通信时，都要在 NAT 路由器上将其本地地址转换成全球 IP 地址，才能和因特网连接。

另外，这种通过使用少量的公有 IP 地址代表较多的私有 IP 地址的方式，将有助于减缓可用的 IP 地址空间的枯竭。在 [RFC](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FRFC%2F10718878)1632 中有对 NAT 的说明。

**二、NAT 的作用**

    NAT 不仅能解决了 lP 地址不足的问题，而且还能够有效地避免来自网络外部的攻击，隐藏并保护网络内部的计算机。

1. 宽带分享：这是 NAT 主机的最大功能。

2. 安全防护：NAT 之内的 PC 联机到 Internet 上面时，他所显示的 IP 是 NAT 主机的公共 IP，所以 Client 端的 PC 当然就具有一定程度的安全了，外界在进行 portscan（端口扫描） 的时候，就侦测不到源 Client 端的 PC 。

**三、NAT 的实现方式**

NAT 的实现方式有三种，即静态转换 Static Nat、动态转换 Dynamic Nat 和端口多路复用 OverLoad。

[**静态**](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E9%259D%2599%25E6%2580%2581)**转换**是指将内部网络的私有 IP 地址转换为公有 IP 地址，IP 地址对是一对一的，是一成不变的，某个私有 IP 地址只转换为某个公有 IP 地址。借助于[静态](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E9%259D%2599%25E6%2580%2581)转换，可以实现外部网络对内部网络中某些特定设备（如服务器）的访问。

**动态转换**是指将内部网络的私有 IP 地址转换为公用 IP 地址时，IP 地址是不确定的，是随机的，所有被授权访问上 Internet 的私有 IP 地址可随机转换为任何指定的合法 IP 地址。也就是说，只要指定哪些内部地址可以进行转换，以及用哪些合法地址作为外部地址时，就可以进行动态转换。动态转换可以使用多个合法外部地址集。当 [ISP](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FISP%2F10152) 提供的合法 IP 地址略少于网络内部的计算机数量时。可以采用动态转换的方式。

**端口多路复用（****Port address Translation,PAT)** 是指改变外出数据包的[源端口](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E6%25BA%2590%25E7%25AB%25AF%25E5%258F%25A3)并进行端口转换，即端口[地址转换](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%259C%25B0%25E5%259D%2580%25E8%25BD%25AC%25E6%258D%25A2)（PAT，Port Address Translation). 采用端口多路复用方式。内部网络的所有[主机](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E4%25B8%25BB%25E6%259C%25BA)均可共享一个合法外部 IP 地址实现对 Internet 的访问，从而可以最大限度地节约 IP 地址资源。同时，又可隐藏网络内部的所有[主机](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E4%25B8%25BB%25E6%259C%25BA)，有效避免来自 internet 的攻击。因此，目前网络中应用最多的就是端口多路复用方式。

**ALG（Application Level Gateway）**，即应用程序级网关技术：传统的 NAT 技术只对 IP 层和传输层头部进行转换处理，但是一些应用层协议，在协议数据报文中包含了地址信息。为了使得这些应用也能透明地完成 NAT 转换，NAT 使用一种称作 ALG 的技术，它能对这些应用程序在通信时所包含的地址信息也进行相应的 NAT 转换。例如：对于 FTP 协议的 PORT/PASV 命令、DNS 协议的 "A" 和 "PTR" queries 命令和部分 ICMP 消息类型等都需要相应的 ALG 来支持。

如果协议数据报文中不包含地址信息，则很容易利用传统的 NAT 技术来完成透明的地址转换功能，通常我们使用的如下应用就可以直接利用传统的 NAT 技术：HTTP、TELNET、FINGER、NTP、NFS、ARCHIE、RLOGIN、RSH、RCP 等。

**四、NAT 的四种类型以及检测方法**

考虑到 UDP 的无状态特性，目前针对其的 NAT 实现大致可分为 Full Cone、Restricted Cone、Port Restricted Cone 和 Symmetric NAT 四种。值得指出的是，对于 TCP 协议而言，一般来说，目前 NAT 中针对 TCP 的实现基本上是一致的，其间并不存在太大差异，这是因为 TCP 协议本身 便是面向连接的，因此无需考虑网络连接无状态所带来复杂性。

**用语定义**

1. 内部 Tuple

：指内部主机的私有地址和端口号所构成的二元组，即内部主机所发送报文的源地址、端口所构成的二元组

**2. 外部 Tuple**：指内部 Tuple 经过 NAT 的源地址 / 端口转换之后，所获得的外部地址、端口所构成的二元组，即外部主机收到经 NAT 转换之后的报文时，它所看到的该报文的源地址（通常是 NAT 设备的地址）和源端口

**3. 目标 Tuple**：指外部主机的地址、端口所构成的二元组，即内部主机所发送报文的目标地址、端口所构成的二元组

**详细释义**

1. Full Cone NAT

：所有来自同一 个内部 Tuple X 的请求均被 NAT 转换至同一个外部 Tuple Y，而不管这些请求是不是属于同一个应用或者是多个应用的。除此之外，当 X-Y 的转换关系建立之后，任意外部主机均可随时将 Y 中的地址和端口作为目标地址 和目标端口，向内部主机发送 UDP 报文，由于对外部请求的来源无任何限制，因此这种方式虽然足够简单，但却不那么安全

2. Restricted Cone NAT

： 它是 Full Cone 的受限版本：所有来自同一个内部 Tuple X 的请求均被 NAT 转换至同一个外部 Tuple Y，这与 Full Cone 相同，但不同的是，只有当内部主机曾经发送过报文给外部主机（假设其 IP 地址为 Z）后，外部主机才能以 Y 中的信息作为目标地址和目标端口，向内部 主机发送 UDP 请求报文，这意味着，NAT 设备只向内转发（目标地址 / 端口转换）那些来自于当前已知的外部主机的 UDP 报文，从而保障了外部请求来源的安 全性

3. Port Restricted Cone NAT

：它是 Restricted Cone NAT 的进一步受限版。只有当内部主机曾经发送过报文给外部主机（假设其 IP 地址为 Z 且端口为 P）之后，外部主机才能以 Y 中的信息作为目标地址和目标端 口，向内部主机发送 UDP 报文，同时，其请求报文的源端口必须为 P，这一要求进一步强化了对外部报文请求来源的限制，从而较 Restrictd Cone 更具安全性

4. Symmetric NAT

：这是一种比所有 Cone NAT 都要更为灵活的转换方式：在 Cone NAT 中，内部主机的内部 Tuple 与外部 Tuple 的转换映射关系是独立于内部主机所发出的 UDP 报文中的目标地址及端口的，即与目标 Tuple 无关； 在 Symmetric NAT 中，目标 Tuple 则成为了 NAT 设备建立转换关系的一个重要考量：只有来自于同一个内部 Tuple 、且针对同一目标 Tuple 的请求才被 NAT 转换至同一个外部 Tuple，否则的话，NAT 将为之分配一个新的外部 Tuple；打个比方，当内部主机以相 同的内部 Tuple 对 2 个不同的目标 Tuple 发送 UDP 报文时，此时 NAT 将会为内部主机分配两个不同的外部 Tuple，并且建立起两个不同的内、外部 Tuple 转换关系。与此同时，只有接收到了内部主机所发送的数据包的外部主机才能向内部主机返回 UDP 报文，这里对外部返回报文来源的限制是与 Port Restricted Cone 一致的。不难看出，如果说 Full Cone 是要求最宽松 NAT UDP 转换方式，那么，Symmetric NAT 则是要求最严格的 NAT 方式，其不仅体现在转换关系的建立上，而且还体现在对外部报文来源的限制方面。

P2P 的 NAT 研究 

第一部分：不同 NAT 实现方法的介绍 

第二部分：NAT 类型检测

**第一部分： 不同 NAT 实现方法的介绍** 

各种不同类型的 NAT(according to RFC) 

_完全圆锥型 NAT(Full Cone NAT):_

内网主机建立一个 UDP socket(LocalIP:LocalPort) 第一次使用这个 socket 给外部主机发送数据时 NAT 会给其分配一个公网 (PublicIP:PublicPort), 以后用这个 socket 向外面任何主机发送数据都将使用这对(PublicIP:PublicPort)。此外，任何外部主机只要知道这个(PublicIP:PublicPort) 就可以发送数据给(PublicIP:PublicPort)，内网的主机就能收到这个数据包 

_地址限制圆锥型 NAT(Address Restricted Cone NAT):_

内网主机建立一个 UDP socket(LocalIP:LocalPort) 第一次使用这个 socket 给外部主机发送数据时 NAT 会给其分配一个公网 (PublicIP:PublicPort), 以后用这个 socket 向外面任何主机发送数据都将使用这对(PublicIP:PublicPort)。此外，如果任何外部主机想要发送数据给这个内网主机，只要知道这个(PublicIP:PublicPort) 并且内网主机之前用这个 socket 曾向这个外部主机 IP 发送过数据。只要满足这两个条件，这个外部主机就可以用自己的 (IP, 任何端口) 发送数据给(PublicIP:PublicPort)，内网的主机就能收到这个数据包 

_端口限制圆锥型 NAT(Port Restricted Cone NAT):_

内网主机建立一个 UDP socket(LocalIP:LocalPort) 第一次使用这个 socket 给外部主机发送数据时 NAT 会给其分配一个公网 (PublicIP:PublicPort), 以后用这个 socket 向外面任何主机发送数据都将使用这对(PublicIP:PublicPort)。此外，如果任何外部主机想要发送数据给这个内网主机，只要知道这个(PublicIP:PublicPort) 并且内网主机之前用这个 socket 曾向这个外部主机 (IP,Port) 发送过数据。只要满足这两个条件，这个外部主机就可以用自己的 (IP,Port) 发送数据给(PublicIP:PublicPort)，内网的主机就能收到这个数据包 

_对称型 NAT(Symmetric NAT):_

内网主机建立一个 UDP socket(LocalIP,LocalPort), 当用这个 socket 第一次发数据给外部主机 1 时, NAT 为其映射一个 (PublicIP-1,Port-1), 以后内网主机发送给外部主机 1 的所有数据都是用这个(PublicIP-1,Port-1)，如果内网主机同时用这个 socket 给外部主机 2 发送数据，第一次发送时，NAT 会为其分配一个(PublicIP-2,Port-2), 以后内网主机发送给外部主机 2 的所有数据都是用这个(PublicIP-2,Port-2). 如果 NAT 有多于一个公网 IP，则 PublicIP-1 和 PublicIP-2 可能不同，如果 NAT 只有一个公网 IP, 则 Port-1 和 Port-2 肯定不同，也就是说一定不能是 PublicIP-1 等于 PublicIP-2 且 Port-1 等于 Port-2。此外，如果任何外部主机想要发送数据给这个内网主机，那么它首先应该收到内网主机发给他的数据，然后才能往回发送，否则即使他知道内网主机的一个(PublicIP,Port) 也不能发送数据给内网主机，这种 NAT 无法实现 UDP-P2P 通信。

**第二部：NAT 类型检测**

前提条件: 有一个公网的 Server 并且绑定了两个公网 IP(IP-1,IP-2)。这个 Server 做 UDP 监听 (IP-1,Port-1),(IP-2,Port-2) 并根据客户端的要求进行应答。

第一步：检测客户端是否有能力进行 UDP 通信以及客户端是否位于 NAT 后？

客户端建立 UDP socket 然后用这个 socket 向服务器的 (IP-1,Port-1) 发送数据包要求服务器返回客户端的 IP 和 Port, 客户端发送请求后立即开始接受数据包，要设定 socket Timeout（300ms），防止无限堵塞. 重复这个过程若干次。如果每次都超时，无法接受到服务器的回应，则说明客户端无法进行 UDP 通信，可能是防火墙或 NAT 阻止 UDP 通信，这样的客户端也就 不能 P2P 了（检测停止）。 

当客户端能够接收到服务器的回应时，需要把服务器返回的客户端（IP,Port）和这个客户端 socket 的 （LocalIP，LocalPort）比较。如果完全相同则客户端不在 NAT 后，这样的客户端具有公网 IP 可以直接监听 UDP 端口接收数据进行通信（检 测停止）。否则客户端在 NAT 后要做进一步的 **NAT 类型**检测 (继续)。

第二步：检测客户端 NAT 是否是 Full Cone NAT？

客户端建立 UDP socket 然后用这个 socket 向服务器的 (IP-1,Port-1) 发送数据包要求服务器用另一对 (IP-2,Port-2) 响应客户端的请求往回 发一个数据包, 客户端发送请求后立即开始接受数据包，要设定 socket Timeout（300ms），防止无限堵塞. 重复这个过程若干次。如果每次都超时，无法接受到服务器的回应，则说明客户端的 NAT 不是一个 Full Cone NAT，具体类型有待下一步检测 (继续)。如果能够接受到服务器从(IP-2,Port-2) 返回的应答 UDP 包，则说明客户端是一个 Full Cone NAT，这样的客户端能够进行 UDP-P2P 通信（检测停止）。

第三步：检测客户端 NAT 是否是 Symmetric NAT？

客户端建立 UDP socket 然后用这个 socket 向服务器的 (IP-1,Port-1) 发送数据包要求服务器返回客户端的 IP 和 Port, 客户端发送请求后立即开始接受数据包，要设定 socket Timeout（300ms），防止无限堵塞. 重复这个过程直到收到回应（一定能够收到，因为第一步保证了这个客户端可以进行 UDP 通信）。 

用同样的方法用一个 socket 向服务器的 (IP-2,Port-2) 发送数据包要求服务器返回客户端的 IP 和 Port。 

比 较上面两个过程从服务器返回的客户端 (IP,Port), 如果两个过程返回的(IP,Port) 有一对不同则说明客户端为 Symmetric NAT，这样的客户端无法进行 UDP-P2P 通信（检测停止）。否则是 Restricted Cone NAT，是否为 Port Restricted Cone NAT 有待检测(继续)。

第四步：检测客户端 NAT 是否是 Restricted Cone NAT 还是 Port Restricted Cone NAT？

客户端建立 UDP socket 然后用这个 socket 向服务器的 (IP-1,Port-1) 发送数据包要求服务器用 IP-1 和一个不同于 Port-1 的端口发送一个 UDP 数据包响应客户端, 客户端发送请求后立即开始接受数据包，要设定 socket Timeout（300ms），防止无限堵塞. 重复这个过程若干次。如果每次都超时，无法接受到服务器的回应，则说明客户端是一个 Port Restricted Cone NAT，如果能够收到服务器的响应则说明客户端是一个 Restricted Cone NAT。以上两种 NAT 都可以进行 UDP-P2P 通信。

注：以上检测过程中只说明了可否进行 UDP-P2P 的打洞通信，具体怎么通信一般要借助于 Rendezvous Server。另外对于 Symmetric NAT 不是说完全不能进行 UDP-P2P 达洞通信，可以进行端口预测打洞，不过不能保证成功。

**五、提升 NAT 类型的好处和方法**

    **提升 NAT 类型的好处：  
**

浏览网页、观看视频、游戏等更顺畅，下载速度更稳定快速，

特别是对那些玩 ED2K/PT 下载、PS4/XBox 主机游戏的，

提升 NAT 类型后更有可能获取到 HigtID、更容易进入游戏房间连线等。

    **提升 NAT 类型的方法**

要提升 NAT 类型，我们必须知道程序显示的 NAT 的 4 个类型的全称：NAT1、NAT2、NAT3、NAT4。

它们分别对应：

**NAT1** → **Full Cone** NAT

**NAT2** → **Address-Restricted Cone** NAT

**NAT3** → **Port-Restricted Cone** NAT

**NAT4** → **Symmetric** NAT

说些比较重要的前话：路由器少一层是一层，这样越有可能得到 NAT1 和 NAT2 这两类 NAT 类型。

一般建议家里的网路是以下两种拓扑类型：

1、光猫桥接→主路由（拨号连接外网用）→副路由（纯 AP 模式，扩展信号）

2、光猫拨号（直接充当主路由）→副路由（纯 AP 模式，扩展信号）

这样的好处是桥接和纯 AP 是不进行 NAT 的，而是 SWitch，所以不会导致多一层 NAT。

如果你的网络是 NAT1，那这是最宽松的网络环境，你想做什么，基本没啥限制；

如果是 NAT4 的话，这是最严格的网络环境，可能会玩不了游戏、下载都没速度；

一般，我们家里的设备都是通过光猫桥接 + 无线路由器拨号的形式连接到外网的，

此时，基本是 NAT2 和 NAT3，正常情况对看网页、游戏及下载都没有过多的限制。

但是，现在个别网络游戏严格要求你的网络环境必须是 “**NAT2**” 以上（NAT2 和 NAT1），才能进行游戏。

而你的网络环境又是 "NAT3” 及 “NAT4", 那到底该怎么办呢？

下面我们介绍一些简单提升 NAT 类型的方法，其实就是进行 [**NAT 穿透**](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fitem%2Fnat%233_3)（[NAT Traversal](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fitem%2Fnat%233_3)）：

1、如果你的路由器有启用 “**Full Cone”、“STUN”、“TURN”、“ICE”、“uPnP”** 等功能，全部都启用了。

2、如果你的路由器没有以上功能，那可以找下有没有 **“**[DMZ](https://link.jianshu.com?t=http%3A%2F%2Fbaike.baidu.com%2Fitem%2Fdmz)**”** 功能（DMZ 是英文 “demilitarized zone” 的缩写，中文名称为“[隔离区](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E9%259A%2594%25E7%25A6%25BB%25E5%258C%25BA)”，也称 “[非军事化区](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E9%259D%259E%25E5%2586%259B%25E4%25BA%258B%25E5%258C%2596%25E5%258C%25BA)”。它是为了解决安装[防火墙](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E9%2598%25B2%25E7%2581%25AB%25E5%25A2%2599)后外部网络的访问用户不能访问内部[网络服务器](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E7%25BD%2591%25E7%25BB%259C%25E6%259C%258D%25E5%258A%25A1%25E5%2599%25A8)的问题，而设立的一个非[安全系统](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%25AE%2589%25E5%2585%25A8%25E7%25B3%25BB%25E7%25BB%259F)与安全系统之间的缓冲区。该缓冲区位于企业内部网络和外部网络之间的小网络区域内。）

有的话，可以启用它，并把你要提升 NAT 类型的主机 IP 地址设置好。

（一般建议有 “Full Cone”、“uPnP” 等，就不要开 "DMZ”了，除非是 PS4/XBox 这类游戏主机要提升 NAT 类型）

3、在 Windows 上把以下三个服务设置为自动启动，并启动该服务：

一般这三个服务都会被奇虎 360 等带启动项优化的软件当做无用启动项**被 “优化”** 成禁止启动。

Function Discovery Provider Host

Function Discovery Resource Publication

SSDP Discovery

4、在 Windows 防火墙，放行你需要提升 NAT 类型的软件或者游戏程序（EXE 程序或者 UWP 程序），

如果你不会放行，也可以直接关闭 Windows 防火墙。（一般不推荐这样做）

第 4 步很重要，这步没做，等于其它的全是在做无用功。

5、如果你的设备是通过电脑共享网络的形式上网的，建议把这个服务也打开：UPnP Device Host

以上，能弄的都弄了，这样你的网络环境就会越好，甚至 NAT1（也就是 Full Cone NAT）都没有问题。