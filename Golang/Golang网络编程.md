![网络编程脑图](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjkou63e46j30u00vodlq.jpg)

## 一、`TCP/IP` 协议簇

### 1. 计算机网络的标准化极其组织

在国际上，负责制定、实施相关网络标准的标准化组织众多，主要有以下几个：

-   **国际标准化组织(`ISO`，`Internationale Organization for Standardization`)**：制定有 `OSI参考模型`、`HDLC` 等；
-   **国际电气电子工程师协会(`IEEE`，`Institute of Electrical and Electronics Engineers`)**：世界上最大的专业技术团体，由计算机和工程学专业人士组成，最著名的研究成果是 [`802标准`](https://zh.wikipedia.org/wiki/IEEE_802)。
-   **国际电信联盟(`ITU`，`International Telegraph and Telephone Consultative Committee`)**：制定了大量有关远程通信的标准。

### 2. 协议

**协议，就是规则的集合。**要在错综负责的网络环境中有条不紊地传输数据，就必须遵守事先约定好的一系列规则，这些规则明确规定了所要交换的数据的格式以及有关同步的问题。这些为了网络中的数据进行交换的规则、约定或者标准，就是我们说的 **网络协议(`Network Protocol`)**。**通过网路协议进行数据交换的双方必须是对等的、水平的实体，不对等的实体之间是没有协议可言的**。比如使用 `TCP/IP` 协议栈进行通信的两个结点，结点`A`的传输层和结点`B`的传输层之间存在协议(`IP协议`)，但是结点`A`的传输层和结点`B`的网络层之间是不存在协议的。

简单来说，**`协议 = 语法 + 语义 + 同步`**。**语法** 规定了传输数据的格式；**语义** 规定了所要完成的功能，即需要发出何种控制信息、完成哪种动作以及针对不同的请求作出何种应答；**同步** 规定了执行各种操作的条件、时序关系。一个完整的协议通常应该具有 **线路管理(建立连接、释放连接等)**、**差错控制**、**数据转换** 等功能。

### 3. `OSI七层模型`

![OSI七层模型](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjkro635u5j31el0u0u0x.jpg)

### 4. `TCP/IP` 模型

对于 `OSI七层模型`，我们认为其底下两层是随系统提供的设备驱动程序和网络硬件，通常情况下，除非需要知道数据链路的某些特性，我们不必去关心这两层的情况；其上面三层可以合并为一层，称为应用层，即web页面、Telnet命令行、FTP服务器等其他网络应用程序所在的层，因为对于 `IP协议` 所在的网络层来说，最上面的三层没有任何区别。这样我们可以看到下面的结构：

![TCP/IP概览](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjkthunzcaj32790u0b2a.jpg)

从字面意义上讲，有人可能会认为 `TCP/IP` 是指 `TCP` 和 `IP` 两种协议。然而在很多情况下，`TCP/IP` 只是利用 `IP` 进行通信时所必须用到的协议群的统称。具体来说，`IP` 或 `ICMP`、`TCP` 或 `UDP`、`TELNET` 或 `FTP`、以及 `HTTP` 等都属于 `TCP/IP` 协议。他们与 `TCP` 或 `IP` 的关系紧密，是互联网必不可少的组成部分。`TCP/IP` 一词泛指这些协议，因此，有时也称 `TCP/IP` 为 **网际协议群**。

下面我们学习 `TCP/IP` 协议簇中四层的细节：

#### 4.1 网络接口层(链路层)

##### 网卡及MAC地址

计算机与外界局域网的连接是通过主机箱内插入的一块 **网络接口板**(又称 **网络适配器`Adapter`**) 或 **网络接口卡**(`Network Interface Card`， `NIC`) 实现的，在这里我们统称为 **网卡**。网卡上装有 **处理器** 和 **存储器**，是工作在数据链路层的网络器件，它是连接计算机实体和传输介质的接口，不仅能实现与局域网传输介质之间的物理连接和电信号匹配，还涉及帧的发送与接收、帧的封装与拆封、数据的编码与解码、数据缓存等功能。

全世界的每一块网卡在出厂时都有一个唯一的代码，我们称为 **介质访问控制(`MAC`，`Media Access Control`) 地址**，这个地址用于控制当前主机在网络上的数据通信。`MAC` 地址长 6 字节，一般使用由连字符或者冒号分割的 6 个十六进制数标识，如 `ac:bc:32:96:86:01`，高 24 位为网卡制造厂商编码，低 24 是该厂商自行分配的网卡序列号。

##### 通信方式

从通信双方信息的交互方式看，可以分为三种基本方式：

1.  **单工**。单工通信只支持信号在一个方向上传输（正向或反向），任何时候不能改变信号的传输方向。例如无线电广播、电视广播等；
2.  **半双工**。半双工通信允许信号在两个方向上传输，但某一时刻只允许信号在一个信道上单向传输。
3.  **全双工**。全双工通信允许数据同时在两个方向上传输，即有两个信道，因此允许同时进行双向传输。

#### 4.2 网际层(网络层)

`IP协议` 是 `TCP/IP` 协议簇的核心协议。正所谓 `Everything over IP`，所有 `TCP`、UDP、ICMP 和 IGMP 数据都通过 `IP` 数据报传输。`IP协议`  提供了一种 **尽力而为**、**无连接** 的数据报交付任务。“尽力而为” 即不保证 `IP` 数据报能成功地到达目的地，任何可靠性必须通过上层协议(如 `TCP`)提供。`IPv4` 和 `IPv6` 都使用这种尽力而为的基本交付模式。“无连接” 是指 `IP协议` 不维护网络单元中数据报相关的任何链接状态信息，**每个数据报相对于其他数据报都是独立的**，这也意味着 `IP` 数据报可以不按发送顺序交付，如果没有时序控制，可能导致错误。

`IP协议` 有 `IPv4` 和 `IPv6`两个版本。其中后者是前者的扩展，在不就的未来，`IPv4` 的地址将耗尽，到时候将会使用 `IPv6`。

>   对应的 RPC文档：[https://tools.ietf.org/html/rfc791](https://tools.ietf.org/html/rfc791)

##### 4.2.1 `IPv4`

**`IPv4`数据报格式**

一个 `IP` 数据报由首部和数据两部分组成，其中首部的前一部分长度固定，共 20 字节，是所有 `IP` 数据报必须具有的；后一部分是一些可选字段，其长度可变，用来提供错误监测和安全等机制。

其组成格式如下：

![IPv4报文格式](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjlav7ah23j30xc0u0x6p.jpg)

解释一下其中字段的含义：

-   **版本(Version)**：4 bits，指 `IP` 的版本，在这里为 4；
-   **首部长度(Header Length)**：4 bits，以 32 位(4 字节)为单位，则最大值为 `(2^4-1) * 4B = 60B`，最小值为 `5 * 4B = 20B`，即没有任何可选字段。
-   **服务类型(Type Of Service)**：8 bits，指明在不同的网络环境下所能提供的服务类型，根据不同的类型选择不同的优先级。(具体细节请自行查看RFC文档)
-   **总长度(Total Length)**：16 bits，包括头部和数据部分的整个数据报长度，单位为 1B，则最大长度为 `2^16 - 1 = 65535B`，用十六进制表示。
-   **标识(Identification)**：16 bits，它是一个计数器，每产生一个数据报就加 1，注意它并不是序号。当一个数据报过大时，必须分片，分片时每个数据报都会复制一次标识位，以便能正确组装原来的数据报。
-   **标志(Flag)**：3 bits，标示分片用。第 0 位时预留位，默认0；第 1 位是 `DF`，`DF = 1` 表示不允许分片，`DF = 0` 表示允许分片；第 2 位时 `MF`，`MF = 1` 表示后面还有分片，`MF = 0` 表示这是最后一个分片。
-   **片位移(Fragment Offset)**：16 bits，表示较长的分组在分片之后，某片在分组中的相对位置。片位移以 8 字节为偏移单位，即每个分片的长度一定是 8B 的整数倍。
-   **生存时间(TTL)**：8 bits，数据报在网络中可通过的路由器数的最大值，表示分组在网络中的寿命，以确保分组不会在网路中循环传送。路由器在转发分组前，先把 `TTL` 减一，当 `TTL = 0` 时，该分组必须被丢弃。
-   **协议(Protocol)**：8 bits，表示此分组携带的数据使用传输层的何种协议，即分组的数据应该交给哪个传输层协议，比较关心的是，6 表示 `TCP`，21 表示 `UDP`。
-   **首部校验和(Header Checksum)**：16 bits，首部校验和只校验分组的首部，而不校验数据部分。
-   **源地址(Source Address)**：32 bits，标识发送方的 `IP` 地址；
-   **目的(Destination Address)**：32 bits，标识接受方的 `IP` 地址。

**`IP` 数据报分片**

一个链路层数据报能承受的最大数据量称为 **最大传送单元(`MTU`)**。因为 `IP数据报` 是被封装在链路层的数据报中，因此链路层的 `MTU` 严格限制着 `IP数据报` 的长度。当 `IP数据报` 的总长度大于链路 `MTU` 时，就需要将 `IP数据报` 中的数据部分，分装在两个或者多个更小的 `IP数据报` 中，这些相对于原来较大的 `IP数据报` 的较小的数据报称为 **`分片`** 。

分片在目的地的网路层会被重新组装，分片的切分和组装需要用到 `IP数据报` 首部的 **标识**、**标志** 和 **片位移** 字段。通过一个例子来说明：

![IP数据报分片](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjlctpypepj32l40u04qp.jpg)

**`IPv4` 地址**

**TODO**

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjld6r6femj305h03e3yx.jpg)

##### 4.2.2 `IPv6`

##### 4.2.3 `ICMP`

#### 4.3 传输层

传输层提供应用进程之间的逻辑通信(注意区分，网络层提供主机与主机之间的逻辑通信)，应用进程之间的通信也称 **端到端的逻辑通信**，这里的 **逻辑通信** 的意思是，传输层之间的通信好像是互相沿着水平方向传送数据，不用去关注下层的实现细节，屏蔽了底层网络核心的细节，但事实上这两个进程之间并没有一条水平方向的物理连接。

**端口** 能够让应用层的各种应用进程将其数据通过端口向下交给传输层，以及让传输层知道应该将报文段中的数据通过哪个端口交付给目的进程。这个端口就是我们平常说的 **段口号**。端口号只具有本地意义，即端口号可以表示本机的应用层的各进程，在因特网的不同计算机的相同端口号是没有任何联系的。段口号的长度为 16 位，能够表示 `2^16` 个不同的端口号。根据端口号的范围，可将其划分为三类：

1.  **周知端口**：`[0, 1023]` 区间内的端口，由 **[IANA](https://www.iana.org/)** 组织同意管理，固定分配给一些非常重要的服务，让所有用户都知道。比如21端口分配给FTP服务，25端口分配给SMTP（简单邮件传输协议）服务，80端口分配给HTTP服务等
2.  **注册端口**：`[1024, 49151]` 区间内的端口，一般由公司或者个人声明、大家熟知的端口，比如 `ElasticSearch` 默认使用 `9200`，`MySQL` 默认使用 `3306`，但是我们可以手动去改变。
3.  **动态端口**：`[49152, 65535]`区间内的端口，这类端口在用户进程运行时会自动选择一个，用完之后释放，下次再使用时再重新分配。

>   你可以在这里找到所有的端口分配：[https://web.archive.org/web/20010604223215/http://www.iana.org/assignments/port-numbers](https://web.archive.org/web/20010604223215/http://www.iana.org/assignments/port-numbers)

再来认识一下 `TCP/IP` 中的 **socket**。在前面的网络层中，我们了解到，网络层提供的是主机到主机之间的逻辑通信，**任何一个接入互联网的主机都有一个唯一的网络地址**，我们称为 `IP 地址`，也就是说，`IP 地址` 可以唯一标识一个网络中的主机。而传输层提供的是应用进程之间的逻辑通信，那如何唯一标识一个主机上的一个应用进程呢？答案就是 `IP + 协议 + 端口`，即 `(IP地址, 协议, 端口)` 这样的三元组就可以唯一标识一个网络中的应用进程。

最后说一下什么是 **面向连接** 与 **面向无连接**。这也是 `TCP` 和 `UDP` 之间的主要区别之一。`TCP` 客户端先与服务端通过 **三次握手** 建立一个连接，之后开始传输数据；而 `UDP` 的客户端与服务端不存在任何的长期关系。举例来说，一个 `UDP` 客户端可以创建一个套接字并发送一个数据给特定的服务器，然后可以立即用同一个套接字发送另一个数据报给另一个服务器；同样地，一个服务器可以用同一个套接字从若干不同的客户接收用户数据报，每一个用户一个数据报。

##### 4.3.1 `UDP`

**`UDP`(`User Datagram Protocol`，用户数据报协议)** 是一个简单的传输层协议，你可以在 [User Datagram Protocol](https://tools.ietf.org/html/rfc768) 找到它的详细说明。应用进程通过套接字写入一个消息，随后该消息被封装到一个 `UDP 数据报` 中，该 `UDP 数据报` 又被封装到一个 `IP 数据报` 中，之后发送到目的地。`UDP` 协议并不保证 `UDP 数据报` 会到达其目的地，也不保证各个数据报的先后顺序在跨网络传输后保持不变，也不保证每个数据只到达一次。

我们看一下一个 `UDP 数据报` 的组成：

![UDP数据报](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjmkoyvezlj31br0u01kx.jpg)

解释一下其中字段的含义：

-   **发送端端口(`Source Port`)**：16 bits，发送端的端口，该字段是可选的，主要用在一些不需要返回的通信中，此时此字段置为0即可。
-   **接收端端口(`Destination Port`)**：16 bits，表示接收端的端口号。
-   **报文总长度(`Length`)**：16 bits，表示`UDP 数据报`首部的长度 和 数据的长度 之和。单位为 `8B`，即一个 `UDP 数据报` 的最大长度是 `2^16 * 8B = 524288B = 512 KB`。当然实际运输时可能不会一次性发送这么大的包，会在网络层装入 `IP 数据报` 时被分片。
-   **校验和(`Checksum`)**：16 bits，其计算方法为：checksum(`UDP 数据报` 中的所有数据 + 一个伪头部 + 为了将 `UDP数据报` 补充成 16bits 的整数倍而补充的0)。

>   下面是伪头部的结构：
>
>   ![伪头部](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjmlfbnqvyj323o0rwnem.jpg)
>
>   其中包含 源IP 和 目的IP，`UDP`协议的代码17，长度字段就是UDP包长度字段值。
>
>   既然是“伪”首部，也就是假的，不仅是“假”首部，而且“假”到连地址空间都没有，其中的信息是从数据报所在IP分组头的分组头中提取的。也就是说伪首部是不占地址空间的，在实际传输中不存在这样的字段。只是在使用的时候把它拿出来一下。
>
>   其次，既然设置了伪首部，那么肯定就是有用的——为了计算检验和！书中原话“其目的是让UDP两次检查数据是否已经正确到达目的地”，具体是那两次呢？我们注意伪首部字段：32位源IP地址、32位目的IP地址、8位协议、16位UDP长度。由此可知，第一次，通过伪首部的IP地址检验，UDP可以确认该数据报是不是发送给本机IP地址的；第二，通过伪首部的协议字段检验，UDP可以确认IP有没有把不应该传给UDP而应该传给别的高层的数据报传给了UDP。
>
>   所以最终逻辑上的 `UDP 数据报` 结构应该是这样的：
>
>   ![UDP数据报](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjmo3eova5j32910u01ky.jpg)

##### 4.3.2 `TCP`



##### 4.3.3 `SCTP`





#### 4.4 应用层

这一层我们只关心 **超文本传输协议`HTTP`**。

##### 4.4.1 概述

`HTTP`(`Hypertext Transfer Protocol`) 超文本传输协议，是在互联网上进行通信时使用的一种协议。最常见的场景是用户与浏览器之间的通信。它属于 应用层 协议，底层使用 `TCP` 进行可靠的传输。

`HTTP` 在传输一段数据时，会以 `流` 的形式将报文数据的内容通过一条已经 “打开”的 `TCP` 传输通道 **按序传输**。传输层的`TCP` 接到上层应用交给它的数据流之后，会按序将数据流打包成一个个的分段，之后交给网络层的 `IP` ，通过网络进行传输。接收端则与这个过程相反，它们将接收到的分段按照顺序组装好，传递给上层的 `HTTP` 进行处理。

###### `URI`

`URI(Uniform Resource Identifier, 统一资源标识符)`，用于标识某一互联网资源，允许用户通过制定的格式访问数据，其格式如下：

```bash
                    hierarchical part
        ┌───────────────────┴─────────────────────┐
                    authority               path
        ┌───────────────┴───────────────┐┌───┴────┐
  abc://username:password@example.com:123/path/data?key=value&key2=value2#fragid1
  └┬┘   └───────┬───────┘ └────┬────┘ └┬┘           └─────────┬─────────┘ └──┬──┘
scheme  user information     host     port                  query         fragment

如：

```



##### 4.4.2 HTTP 1.1

##### 4.4.3 HTTP 2.0

##### 4.4.4 HTTP 3.0

##### 4.4.5 HTTPS

## 二、`Socket` 编程

### 1. 什么是 `socket` 编程

使用 `TCP/IP` 协议的应用程序通常采用应用编程接口：**UNIX BSD的套接字(socket)** 和 **UNIX System V的TLI**（已经被淘汰），来实现网络进程之间的通信。就目前而言，几乎所有的应用程序都是采用 `socket`。`socket` 起源于 `Unix`，而 `Unix/Linux` 基本哲学之一就是**“一切皆文件”**，都可以用`open –> write/read –> close`模式来操作。`socket` 就是该模式的一个实现，即 `socket` 是一种特殊的文件，一些 `socket` 函数就是对其进行的操作(读/写IO、打开、关闭)。

如果要给 `socket` 一个定位，那应该是：`socket` 就像是一组 `接口(interface)`，它将更复杂的 `TCP/IP` 协议簇隐藏在 `socket` 接口后面，只对用户暴露更简单的接口，就像操作系统隐藏了底层的硬件操作细节而只对用户程序暴露接口一样，它是 应用层 与 `TCP/IP`协议簇 通信的中间软件抽象层。可以用一张图来表示这个关系：

![socket的定位](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjmnmaizsaj31c10u0e3v.jpg)

而整个 `socket` 工作流程可以用下面这张图来表示：

![socket工作流程](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjmqesnq9xj30u01g2b21.jpg)

如果用语言描述的话，服务端首先初始化 `socket`，然后与端口绑定(`bind`)，并对端口进行监听(`listen`)，阻塞(`accept`) 等待客户端连接；在客户端这边，先初始化一个 `socket`，通过 **TCP 三次握手** 与服务端简历连接(`connect`)，服务端接收请求(`read`)并处理请求，然后将结果返回客户端(`write`)，客户端读取数据(`read`)，最后关闭连接(`close`)，一次通信结束。

接下来我们详细看看这几个函数：

### 2. `socket` 系列函数

#### 2.1 `socket()`

为了执行 `网络I/O`，一个进程第一件事就是调用 `socket()` 函数，传入期望的 **通信协议类型**，得到一个 **套接字描述符**：

```c
int socket(int family, int type, int protocol);  // 如果创建成功，会返回非负整数 sockfd，即套接字描述符 
```

其中 `family` 指明 **协议簇**：

|  `family`  |     说明      |
| :--------: | :-----------: |
| `AF_INET`  |  `IPv4` 协议  |
| `AF_INET6` |  `IPv6` 协议  |
| `AF_LOCAL` | `Unix` 域协议 |
| `AF_ROUTE` |  路由套接字   |
|  `AF_KEY`  |  密钥套接字   |

`type` 指明 **套接字类型**：

|      `type`      |      说明      |
| :--------------: | :------------: |
|  `SOCK_STREAM`   |  字节流套接字  |
|   `SOCK_DGRAM`   |  数据报套接字  |
| `SOCK_SEQPACKET` | 有序分组套接字 |
|    `SOCK_RAW`    |   原始套接字   |



`protocol` 指 **传输层协议**（已经废弃，因为前两个参数就能唯一确定使用什么协议，因此这个参数一般写成 0）：

|   `protocol`   |      说明       |
| :------------: | :-------------: |
| `IPPROTO_TCP`  | `TCP` 传输协议  |
| `IPPROTO_UDP`  | `UDP` 传输协议  |
| `IPPROTO_SCTP` | `SCTP` 传输协议 |

但并非所有的 `family` 和 `type` 的组合都是有效的，下表给出有效的组合以及对应的具体协议：

![family和type组合](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjmw3h9xkpj30kn0bewfe.jpg)

>   “是” 表示该组合也是有效的，但是目前没有现成的协议供使用。空白则表示 非有效组合。
>
>   `socket()` 函数在成功时会返回一个小的非负整数值，它与文件描述符类似，我们把它称为 **套接字描述符(`socket descriptpr`,  `sockfd`)**，这个描述符唯一描述一个 `socket`。

#### 2.2 `bind()`

`bind()` 将一个本地协议地址赋予一个套接字。对于网际协议，协议地址是 32 位的`IPv4` 地址 或 128 为的 `IPv6` 地址与 16 位的 `TCP` 或 `UDP` 端口号的组合：

```c
int bind(int sockfd, const struct sockaddr *myaddr, socklen_t addrlen); // 返回： 0成功，-1失败
```

`sockfd` 是由 `socket()` 函数返回的套接字描述符。

`myaddr` 是一个指向特定协议的地址结构的指针，表示绑定给 `sockfd` 的协议地址。这个地址结构根据地址创建socket时的地址协议族的不同而不同，比如 `IPv4` 对应的是：

```c
struct sockaddr_in {
    sa_family_t    sin_family; /* Address Family: AF_INET */
    in_port_t      sin_port;   /* Port Number */
    struct in_addr sin_addr;   /* Internet Address */
};

/* Internet address. */
struct in_addr {
	__be32	s_addr;
};
```

`IPv6` 对应的是：

```c
struct sockaddr_in6 {
	unsigned short int	sin6_family;    /* AF_INET6 */
	__be16			sin6_port;          /* Port Number # */
	__be32			sin6_flowinfo;      /* IPv6 flow information */
	struct in6_addr		sin6_addr;      /* IPv6 address */
	__u32			sin6_scope_id;      /* scope id (new in RFC2553) */
};

/*
 *	IPv6 address structure
 */
struct in6_addr {
	union {
		__u8		u6_addr8[16];
		__be16		u6_addr16[8];
		__be32		u6_addr32[4];
	} in6_u;
};
```

`Unix 域` 对应的是：

```c
#define UNIX_PATH_MAX	108

struct sockaddr_un {
	__kernel_sa_family_t sun_family; /* AF_UNIX */
	char sun_path[UNIX_PATH_MAX];	/* pathname */
};
```

`addrlen` 是第二个参数地址结构的长度。正如大多数 `socket` 接口一样，内核不关心地址结构，当它复制或传递地址给驱动的时候，它依据这个值来确定需要复制多少数据。这已经成为 `socket` 接口中最常见的参数之一了。换一种方式理解，第二个参数是通用地址格式 `sockaddr * addr`，实际传入的参数可能是IPv4、IPv6 或者本地套接字格式，`bind()` 函数会根据第三个参数 `addrlen` 来判断传入的 `addr` 应该怎么解析。

对于 `bind()` 函数中的 `IP` 和 端口，不管是服务端还是客户端，都可以指定，也都可以不指定，也都可以指定其中一个，只不过会有不同的效果。

对于服务器而言，一般情况下要指定 `IP` 和 端口，因为它是提供服务的一方，必须明确自己的身份，让客户端能够在网络中找到，即通过之前说的三元组 `(IP地址, 协议, 端口)` 来唯一标识这个进程(服务)。不过，服务器进行 `IP` 绑定的时候，有 具体的 `IP` 和 通配 `IP` 之分，前者表示只接收来自该 `IP` 的连接，后者表示可以接受来自任何 `IP` 的连接，来者不拒。而服务器启动的时候的端口，一般都是众知端口，如果没有指定，`connect()` 或者 `listen` 的时候内核会为其指定一个。

对于客户端而言，没有必要去指定端口，内核会选择一个可用的动态端口。如果客户端指定了 `IP` 地址，则相当于为发送出去的 `IP数据报` 分配了源IP地址，但这时这个`IP` 地址必须属于这个主机，不能分配一个不存在的 `IP`。

简而言之，采用 `TCP` 或者 非对等的 `UDP` 通信时，客户端不需要 `bind()` 他自己的 `IP` 和端口号，如果需要的话，内核会确定源 `IP` 地址并选择一个临时端口作为源端口，而服务器必须要 `bind()` 自己本机的 `IP` 和端口号。

我们来看一个初始化 `IPv4 TCP` 套接字的实例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/socket.h>
#include <netinet/in.h>
int make_socket (uint16_t port)
{
  int sock;
  struct sockaddr_in name;
  /* 创建字节流类型的 IPV4 socket. */
  sock = socket (PF_INET, SOCK_STREAM, 0);
  if (sock < 0)
    {
      perror ("socket");
      exit (EXIT_FAILURE);
    }
  /* 绑定到 port 和 ip. */
  name.sin_family = AF_INET; /* IPV4 */
  name.sin_port = htons (port);  /* 指定端口 */
  name.sin_addr.s_addr = htonl (INADDR_ANY); /* 通配地址 */
  /* 把 IPV4 地址转换成通用地址格式，同时传递长度，并进行 bind 操作 */
  if (bind (sock, (struct sockaddr *) &name, sizeof (name)) < 0)
    {
      perror ("bind");
      exit (EXIT_FAILURE);
    }
  return sock
}
```



#### 2.3 `connect()`

`TCP` 客户端使用 `connect()` 函数用来建立与服务端的连接：

```c
int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);  // 0成功 -1出错
```

参数描述与 `bind()` 函数一样，不过这个时候，套接字地址结构必须含有服务器的 `IP` 地址 和 具体的端口号。

客户在调用函数 `connect()` 前不必非得调用 `bind()` 函数，因为如果需要的话，内核会确定源 IP 地址，并按照一定的算法选择一个临时端口作为源端口。

如果是 `TCP` 套接字，调用 `connect()` 会激发 `TCP` 的 **三次握手** 过程，而且仅在连接成功或者出错时才返回。

#### 2.4 `listen()`

```c
int listen(int sockfd, int backlog);  // 0成功 -1出错
```

`listen()` 函数仅由服务器调用，它做两件事：

1.   当我们用 `socket()` 创建一个 套接字 时，它默认是一个主动套接字，即它默认是一个将要调用 `connect()` 对服务器发起连接的客户端套接字。而 `listen()` 函数会把一个这样的主动套接字变成一个被动套接字，指示内核接受指向该套接字的。连接要求。对外表现效果就是，`TCP 状态转移图` 中，`listen()` 会将 `CLOSED 状态` 转换成 `LISTEN 状态`。
2.  第二个参数指定了内核应该为响应的套接字可以排队的最大连接个数。

>   当进程在处理一个个连接请求的时候，有可能还存在其他的链接请求，因为 `TCP` 连接有一个三次握手的过程，并非原子性的，所以就有可能存在一种半连接状态。这时就有可能出现尝试连接的用户数太多、服务器无法快速完成连接的情况。为了解决这个问题，内核会在自己的进程空间中维护两个队列，而且这两个队列总长度不能超过限定值 `backlog`：
>
>   -   未完成连续队列：在三次握手期间，客户端发送一个 `SYN` 并到达服务器请求建立连接，而服务器正在等待完成该过程；
>   -   已完成连续队列：三次握手完成后 或者 连接超时，该连接就会被从 **未完成连续队列** 转移到 **已完成连续队列** 的末尾，当进程调用 `accept()` 时，会从队列头部返回一个连接出去。如果当前队列为空，那么进程将被置于休眠状态，直到此队列中有数据。
>
>   当然，内核空间有限，我们不能随意指定一个 `backlog` 值，一般在 30 以内。

#### 2.5 `accept()`

TCP服务器端依次调用socket()、bind()、listen()之后，就会监听指定的socket地址了。TCP客户端依次调用socket()、connect()之后就想TCP服务器发送了一个连接请求。TCP服务器监听到这个请求之后，就会调用accept()函数取接收请求，这样连接就建立好了。之后就可以开始网络I/O操作了，即类同于普通文件的读写I/O操作：

```c
int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);  // 成功则返回非负的已连接套接字，出错返回-1
```

`accept()` 主要用在基于连接的套接字类型，比如 `TCP` 或者 `SCTP` 。它从 `listen()` 的 **已完成连续队列** 中取出队首的一个连接，创建一个不同于 `socket()` 创建的(称为 **监听套接字(`listening socket`)**)新的套接字——**已连接套接字(`connected socket`)**，并返回指向该套接字的文件名描述符。

>   需要区分这两个套接字。一般情况下，服务器仅仅创建一个监听套接字，它在该服务的生命周期内一直存在；内核为每一个建立连接的客户创建一个新的套接字，当服务完成后，这个新创建的 **已连接套接字** 就会被关闭，对应的就是 `TCP连接` 的释放。

`accept()` 成功则返回 **已连接套接字** 的描述符，失败则返回 -1。另外需要注意后两个参数，传输的是指针，这其实也是返回值，因为 `C 语言` 中函数没有多返回值，因此采用这种直接修改指针的方式复制，他们分别代表 客户端进程的协议地址、指向该地址的大小。如果我们对客户端协议不感兴趣，可以将这两个指针都置空。

#### 2.6 `read()` 、 `write()`

经过上面的几个步骤，客户端与服务端已经建立了连接，此时可以调用 `网络I/O` 进行读写操作，即实现不同网络中不同进程之间的通信。

```c
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
```

一般也会用到以下几组函数：

>   -   `read()` / `write()`
>   -   `recv()` / `send()`
>   -   `readv()` / `writev()`
>   -   `recvmsg()` / `sendmsg()`
>   -   `recvfrom()` / `sendto()`

#### 2.7 `close()`

通常情况下，`Unix` 系统中 `close()` 也用来关闭套接字，并终止 `TCP` 连接：

```c
int close(int sockfd); 
```

`close()` 将一个套接字标记为 **已关闭**，然后立即返回到调用进程，被关闭的套接字描述符不能再由进程使用，也就是说它不能再作为 `read()` 或 `write()` 的第一个参数。但是 `TCP` 的连接并不是立即关闭的，因为此时有可能出现 已经存在于 `TCP` 发送队列的数据，等发送完毕后才会去执行 `TCP` 的关闭流程。

#### 2.8 并发服务器的底层设计

我们先了解一下 `Unix` 系统中的 `fork()` 函数，它是 `Unix` 系统中 **派生出新进程** 的 **唯一** 方法：

```c
pid_t fork(void);
```

该函数的难以理解之处在于 **调用一次，返回两次**。它在调用进程(父进程)中返回一次，返回值是新派生出的进程(子进程)的进程 `PID` ；在子进程中又返回一次，返回值为 0 。因此可以根据返回值来判断当前进程时父进程还是子进程。

>   任何一个子进程都只有一个父进程，而一个父进程可以有多个子进程。子进程可以通过库函数 `getppid()` 获取父进程的 `PID` ，但是父进程无法获取各个子进程的 `PID` 。

一个进程调用 `fork()` 函数后，系统先给新的进程分配资源，例如存储数据和代码的空间。然后把原来的进程的栈段所有值都复制到新的新进程中，只有少数值与原来的进程的值不同。相当于克隆了一个自己。需要注意的一点：就是调用 `fork()` 之后，两个进程同时执行的代码段是 `fork()` 函数之后的代码，而之前的代码已经由父进程执行完毕。看一个 `demo`：

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    int forkid;
    int count = 0;
    printf("fork 之前, count: %d\n", count);

    forkid = fork();  // fork 出子进程

    if (forkid == 0)
    {
        // 子进程
        printf("我是子进程\t");
        count++;
        printf("子进程中 count: %d\n", count);
    }
    else
    {
        // 父进程
        printf("我是父进程\t");
        count++;
        printf("父进程中 count: %d\n", count);
    }

    count++;

    printf("主进程中 count: %d\n", count);

    return 0;
}

// 输出结果：
/*
fork 之前, count: 0
我是父进程      父进程中 count: 1
主进程中 count: 2
我是子进程      子进程中 count: 1
主进程中 count: 2
*/
```

父进程在调用 `fork()` 之前打开的所有文件描述符在 `fork()` 之后是和子进程共享的，而目前大多数网络服务器正是利用了这个特性来做到并发：父进程 `accept()` 之后 `fork()` 出一个子进程，此时二者共享之前已经连接好的套接字，子进程也持有一个该连接的套接字，这个时候父进程可以关掉当前的连接，继续接收另一个连接。这个过程可以用下面这个伪代码来描述：

```c
int main()
{
    int listenfd, connfd;

    listenfd = socket();   // 创建一个全局的 监听套接字
    bind(listenfd, ...);   // bind ...
    listen(listenfd, ...); // listen ...

    for (;;)
    {
        connfd = accept(listenfd, ...) ； // 接收一个客户端的连接

        if (fork() == 0)
        {
            // 启动一个子进程去处理客户端连接
            close(listenfd);         // 子进程中持有的是listenfd的副本，没有用到，关掉
            process_request(connfd); // 子进程中处理客户端请求
            close(connfd);           // 返回后关闭与该客户端的连接
            exit(0);                 // 子进程退出
        }
        close(connfd);  // 父进程没必要保留某个连接的文件描述符，因为子进程已经在处理了，所以关掉减少引用计数
    }
}
```

>   关于 **引用计数**：在  `Unix` 系统中，每一个文件或者套接字都有一个 引用计数，维护在文件表项中，保存着 **当前打开着的 引用该文件或套接字的描述符 的个数**。垃圾清理和资源回收只会清理引用计数值为 0 的对象。在上面的伪代码中，`socket()` 返回后使得 `listenfd` 的引用计数值变为 1，`accept()` 返回后 `connfd` 的引用计数值变为 1；`fork()` 返回后，这两个描述符在父进程和子进程中共享，子进程中多了一份拷贝，因此 `listenfd` 和 `connfd` 各自变成 2。这样一来，当第 21 行父进程关闭 `connfd` 时，只是将其引用计数值减 1，结果仍为一个大于 0 的数，所以子进程中可以继续对其进行读写操作；同样，在子进程中第 16 行，子进程中用不到父进程中的 `listenfd` ，所以将其关掉，使得 `listenfd` 的引用计数始终保持在 1，这样也方便进行资源回收。

我们详细了解一下上面伪代码的过程：

首先，客户端发起请求，服务端调用 `listen()` 后 `accept()` 之前的状态如下：

![accept() 前状态](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjnxr0519zj31h80gkaek.jpg)

`accept()` 后，连接被服务器内核接受，新的已连接套接字 `connfd` 被创建，客户端和服务端可以通过这个跨连接套接字读写数据：

![accept()被接受后](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjnxv199nvj31hc0fkdl8.jpg)

并发服务器 `fork` 出一个子进程专门处理这个客户端连接：

![fork() 之后](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjnxyfvi7hj31ll0u0wou.jpg)

此时 `listenfd` 和 `connfd` 在父进程和子进程之间进行共享，下一步，父进程关闭 `connfd` ，子进程关闭 `listenfd`，然后由子进程监控 已连接套接字`connfd`，而父进程可以继续等到接受来自其他客户端的请求：

![子进程掌管与客户端的连接](https://tva1.sinaimg.cn/large/007S8ZIlgy1gjny17xi16j31ll0u0gva.jpg)

### 3. 从 `socket` 角度看三次握手四次挥手

先看一张图: ![](https://hiberabyss.github.io/2018/03/14/unix-socket-programming/tcp_open_close.jpg)

TCP 状态转换: ![](https://hiberabyss.github.io/2018/03/14/unix-socket-programming/tcp_status1.jpg)

### 4. 本地套接字

本地套接字是 `IPC(InterProcess Communication，进程间通信)` 的一种方式，常用的方式有管道、共享消息队列，也包括此处说的 `本地套接字`。

本地套接字一般也叫 `Unix域套接字(Unix Domain Socket)`，最新的规范已经改名为 `本地套接字`。它是一种特殊的套接字，与 `TCP/IP` 套接字不同。`TCP/UDP` 即使在本地地址通信，也要走系统网络协议栈，而本地套接字，严格意义上说提供了一种单主机跨进程间调用的手段，主要用于一台主机的进程间通信，不需要网络协议，是基于文件系统的。也正是因为不需要经过网络协议栈，所以也不需要打包拆包、计算校验和、维护序号和应答等，只是将应用层数据从一个进程拷贝到另一个进程，这是因为，IPC 机制本质上是可靠的通讯，而网络协议是为不可靠的通讯设计的，效率比`TCP/UDP` 套接字都要高许多。类似的 `IPC机制` 还有 `UNIX管道`、`共享内存` 和 `RPC调用` 等。

与 `Internet Domain Socket`类似，其接口是一致的，可以支持 `字节流(SOCK_STREAM)`和 `数据报(SOCK_DGRAM)` 两种协议。不过，本地套接字不需要 `IP` 和 `Port`，而是通过一个文件名对应的文件来表示。

我们通过一个小 demo 来观察一下：

首先，服务端我用 `golang` 来实现：

```go
import (
	"bufio"
	"fmt"
	"io"
	"log"
	"net"
	"os"
	"time"
)

/*
* @CreateTime: 2021/3/23 14:56
* @Author: hujiaming
* @Description:
 */

func main() {
    // 建议这里设置成绝对路径 
	socketFileName := "/tmp/unix_domain.sock1"
	var unixAddr *net.UnixAddr
	unixAddr, _ = net.ResolveUnixAddr("unix", socketFileName)
again:
	lis, err := net.ListenUnix("unix", unixAddr)
	if err != nil {
		log.Print("listen to Unix Domain Socket error: ", err, "retrying...")
		// 此处可能是 socket 文件已经存在，尝试移除后重新连接
		err = os.Remove(socketFileName)
		if err != nil {
			log.Panic("retry to connect failed")
		}
		goto again
	}
	defer lis.Close()

	for {
		conn, err := lis.AcceptUnix()
		if err != nil {
			log.Print("accept error", err)
			continue
		}
		log.Println("A client connected : " + conn.RemoteAddr().String())
		go process1(conn)
	}
}

func process1(conn *net.UnixConn) {
	ipStr := conn.RemoteAddr().String()
	defer func() {
		fmt.Println("disconnected :" + ipStr)
		conn.Close()
	}()
	reader := bufio.NewReader(conn)

	for {
		message, err := reader.ReadString('\n')  // 通过 `\n` 作为消息的分隔符
		if err != nil {
			log.Print("readString error ", err)
			return
		}

		fmt.Println(string(message))
		msg := time.Now().String() + "\n"
		b := []byte(msg)
		conn.Write(b)
	}
}
```

为了体现跨进程通信，客户端我用 python 实现：

```python

import socket

socket_file = "/tmp/unix_domain.sock1"

def run():
    sock = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
    sock.connect(socket_file)
    while True:
        data = input("请输入您需要发送的数据:")
        if data == "quit":
            break
        data = data + "\n"  # \n 是给 go 服务端读的
        sock.send(data.encode("utf-8"))
        receive = sock.recv(1024)  # 多读一些数据，不影响的
        print("py : ", receive.decode("utf-8"))
    sock.close()


if __name__ == '__main__':
    run()

```

当然，golang 的客户端也很简单：

```go

import (
	"bufio"
	"fmt"
	"log"
	"net"
	"os"
)

/*
* @CreateTime: 2021/3/23 15:05
* @Author: hujiaming
* @Description:
 */

func main() {
	file := "/Users/hujiaming/go/src/hujm.net/tcpip_demo/unixsocket/unix_domain.sock1"
	conn, err := net.Dial("unix", file) // 发起请求
	if err != nil {
		log.Fatal(err) // 如果发生错误，直接退出程序，因为请求失败所以不需要 close
	}
	defer conn.Close() 

	input := bufio.NewScanner(os.Stdin) // 创建 一个读取输入的处理器
	reader := bufio.NewReader(conn)     // 创建 一个读取网络的处理器
	for {
		fmt.Print("请输入需要发送的数据: ")       // 打印提示
		input.Scan()                    // 读取终端输入
		data := input.Text()            // 提取输入内容
		conn.Write([]byte(data + "\n")) // 将输入的内容发送出去，需要将 string 转 byte 加 \n  作为读取的分割符

		msg, err := reader.ReadString('\n') // 读取对端的数据
		if err != nil {
			log.Println("readString ", err)
		}
		fmt.Println(msg) // 打印接收的消息
	}
}

```

### 5. 关于 TIME_WAIT

在关闭一个 TCP连接 时，会经过一个 **四次握手** 的过程，在这个过程中，发起断开连接的一方会有一段时间处于 `TIME_WAIT` 状态。本节尝试说明这一现象出现的前因后果以及对应的解决方案。

#### TIME_WAIT 出现的场景

下图是 **四次挥手** 的流程图：

![四次挥手](/Users/hujiaming/Library/Application%20Support/typora-user-images/image-20210323185014383.png)

主机 1 主动发起连接终止操作，先发送 FIN 报文，自己状态变为 `FIN_WAIT_1`；主机2 收到后进入 `CLOSE_WAIT` 状态，同时回复一个 `ACK` 应答，主机1 收到应答后状态变为 `FIN_WAIT_2`；发送 FIN 代表当前端再没有数据发送了，但仍旧可以接收数据，也就是说，主机1 进入 `FIN_WAIT_2` 后，依旧可以接收来自主机2 的数据；当主机2 没有数据再发送时，向主机1 发送 FIN 报文，主机2 进入 `LAST_ACK` 状态，主机1 收到来自对端的 FIN 报文后，就进入了 `TIME_WAIT` 状态。

主机1 在  `TIME_WAIT` 的停留时间是固定的，为 `MSL(Maximum Segment Lifetime，最长TCP分段生命时期)` 的两倍，一般称之为 `2MSL`。在 Linux 系统中，这个值是固定的 60s。等过了这个时间之后，主机1 就会进入 `CLOSED` 状态。需要记住的是：**只有发起终止的一方才会有 `TIME_WAIT` 状态。**

>   关于 `MSL`：MSL 是任何 IP 数据报能够在因特网中存活的最长时间。其实它的实现不是靠计时器来完成的，在每个数据报里都包含有一个被称为 `TTL（time to live）` 的 8 位字段，它的最大值为 255。`TTL` 可译为 **“生存时间”**，这个生存时间由源主机设置初始值，它表示的是一个 IP数据报可以经过的最大跳跃数，每经过一个路由器，就相当于经过了一跳，它的值就减 1，当此值减为 0 时，则所在的路由器会将其丢弃，同时发送 `ICMP` 报文通知源主机。[RFC793](https://tools.ietf.org/html/rfc793) 中规定 `MSL` 的时间为 2 分钟，`Linux` 实际设置为 30 秒。

#### TIME_WAIT 的作用

为什么不直接进入 `CLOSED` 状态，非要停留在 `TIME_WAIT` 这个状态呢？为什么非要等两个 MSL ？有两个方面的原因。

原因一，为了确保最后的 ACK 能被动方接收，从而正确关闭。设想以下场景：主机2 只有在收到 ACK 后才会进入 `CLOSED` 状态，如果预设的时间内没有收到，会重发 FIN；如果主机1 没有维护 `TIME_WAIT` 状态而直接关闭，当再次收到主机2 的 FIN 时，因为已经丢失上下文，只能回复一个 `RST` 报文，这将导致主机2 关闭时出现错误。当主机1 有 `TIME_WAIT` 状态时，就可以在再次接收到 FIN 时回复一个 ACK，使得主机2 进入正常的 `CLOSED` 状态。

原因二，为了让之前连接实例的过期报文自然消失从而不影响下次连接。在复杂的网络环境中，由于某些原因，某一方突然断开了连接，或者由于 crash 造成之前的连接信息全部丢失，这种情况下，需要重新建立连接(用新的 ISN)。但另一方并不知道这一方面已经 crash 的事实，可能继续用上一次的连接去发送数据，crash 的一方收到这个上一次连接的数据之后，只能将其丢弃，并重新发起建立连接的请求(三次握手)。我们考虑这样一个场景，在原链接中断之后，又重新创建了一个原连接的 “clone”，说是“clone”是因为这个连接和原来连接的四元组完全相同(不同的是各自三次握手时的初始序列号)，如果上一个连接的“迷失报文”经过一段时间后到达另一方，要么这个“迷失报文”的序列号正好也在新建立的序列号窗口之中，被另一方接收，会对正常的数据流造成影响，要么这个“迷失报文”的序列号不在接收窗口范围之中，被丢弃(这是理想结果)。所以，TCP 就设计出了这么一个机制，经过 2MSL 这个时间，足以让两个方向上的分组都被丢弃，使得原来连接的分组在网络中都自然消失，再出现的分组一定都是新化身所产生的。

划重点，`2MSL` 的时间是从 主机1 接收到 `FIN` 后发送 `ACK` 开始计时的；如果在 `TIME_WAIT` 时间内，因为 主机1 的 `ACK` 没有传输到主机2，主机 1 又接收到了主机 2 重发的 `FIN` 报文，那么 `2MSL` 时间将重新计时。道理很简单，因为 `2MSL` 的时间，目的是为了让旧连接的所有报文都能自然消亡，现在主机 1 重新发送了 `ACK` 报文，自然需要重新计时，以便防止这个 `ACK` 报文对新可能的连接化身造成干扰。

总结：** `TIME_WAIT` 的引入是为了让 `TCP` 报文得以自然消失，同时为了让被动关闭方能够正常关闭。**

#### TIME_WAIT 的危害

主要危害是对 **端口资源** 的占用。一个 TCP 连接至少消耗一个本地端口，而本地端口是有限的(一般可开启的端口范围是 32768～61000)，如果 `TIME_WAIT` 过多，端口被耗尽，那新的连接就无法创建，整个服务对外表现为不可用，放在生产环境下妥妥的一个“事件”~

#### 如何优化TIME_WAIT

Linux 下，有 `net.ipv4.tcp_tw_reuse` 这个选项：

```bash
net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
```

这个参数只对于发起连接的一方有用(即客户端)，并且对应的 TIME_WAIT 状态的连接创建时间超过 1 秒才可以被复用。

其他方案都有一定的风险，不建议使用，这里也不罗列了。

### 6. 如何正确地关闭连接

在绝大数情况下，TCP 连接都是先关闭一个方向，此时另外一个方向还是可以正常进行数据传输。举个例子，客户端主动发起连接的中断，将自己到服务器端的数据流方向关闭，此时，客户端不再往服务器端写入数据，服务器端读完客户端数据后就不会再有新的报文到达。但这并不意味着，TCP 连接已经完全关闭，很有可能的是，服务器端正在对客户端的最后报文进行处理，比如去访问数据库，存入一些数据；或者是计算出某个客户端需要的值，当完成这些操作之后，服务器端把结果通过套接字写给客户端，我们说这个套接字的状态此时是“半关闭”的。最后，服务器端才有条不紊地关闭剩下的半个连接，结束这一段 TCP 连接的使命。

然而，上面的场景是在理想环境下的，比较“优雅”，如果服务器端处理不好，就会导致最后的关闭过程是“粗暴”的，达不到我们上面描述的“优雅”关闭的目标，形成的后果，很可能是服务器端处理完的信息没办法正常传送给客户端，破坏了用户侧的使用场景。

先看看关闭连接的方式：

#### close()

```c
int close(int sockfd)
```

对 `连接套接字sockfd` 执行 `close` 操作，成功返回 0，失败返回-1。

这个函数会对套接字引用计数减一，系统一旦发现套接字引用计数到 0，就会对套接字进行彻底释放，并且会关闭TCP 两个方向的数据流。

什么是 **套接字引用计数**？因为套接字可以被多个进程共享，我们通过 fork 的方式产生子进程，套接字引用次数 +1；调用一次 `close` 函数，套接字引用次数 -1。这就是套接字引用计数的含义。

`close()` 函数具体是如何关闭两个方向的数据流呢？在输入方向，系统内核会将该套接字设置为 **不可读**，任何读操作都会返回异常；在输出方向，系统内核尝试将发送缓冲区的数据发送给对端，并最后向对端发送一个 `FIN` 报文，接下来如果再对该套接字进行写操作会返回异常。

如果对端没有检测到套接字已关闭，还继续发送报文，就会收到一个 RST 报文，告诉对端：“Hi, 我已经关闭了，别再给我发数据了。”

使用 `close()` ，要注意两点：

-   `close()` 函数只是把套接字引用计数减 1，未必会立即关闭连接；
-   `close()` 函数如果在套接字引用计数达到 0 时，立即终止读和写两个方向的数据传送。

我们会发现，close 函数并不能帮助我们只关闭连接的一个方向，那么如何在需要的时候关闭一个方向呢？试试 `shutdown()` 函数。

#### shutdown()

```c
int shutdown(int sockfd, int howto)
```

对 `连接套接字sockfd` 执行 shutdown 操作，若成功则为 0，若出错则为 -1。

`haoto` 有三个选项：

-   `SHUT_RD(0)`：关闭连接的“读”这个方向，对该套接字进行读操作直接返回 `EOF`。从数据角度来看，套接字上接收缓冲区已有的数据将被丢弃，如果再有新的数据流到达，会对数据进行 `ACK`，然后悄悄地丢弃。也就是说，对端还是会接收到 `ACK`，在这种情况下根本不知道数据已经被丢弃了。
-   `SHUT_WR(1)` ：关闭连接的“写”这个方向，这就是常被称为”半关闭“的连接。此时，不管套接字引用计数的值是多少，都会直接关闭连接的写方向。套接字上发送缓冲区已有的数据将被立即发送出去，并发送一个 FIN 报文给对端。应用程序如果对该套接字进行写操作会报错。
-   `SHUT_RDWR(2)` ：相当于 SHUT_RD 和 SHUT_WR 操作各一次，关闭套接字的读和写两个方向。

#### close() 和 shutdown() 的差别

差别主要有三点：

1.  `close()` 会关闭连接，并释放所有连接对应的资源，而 `shutdown()` 并不会释放掉套接字和所有的资源；
2.  `close()` 存在引用计数的概念，并不一定导致该套接字不可用；``shutdown()`` 则不管引用计数，直接使得该套接字不可用，如果有别的进程企图使用该套接字，将会受到影响;
3.  `close()` 的引用计数导致不一定会发出 `FIN` 结束报文，而 `shutdown()` 则总是会发出 `FIN` 结束报文，这在我们打算关闭连接通知对端的时候，是非常重要的。

通过 golang 代码来体现：

服务端代码：

```go
package main

/*
* @CreateTime: 2021/3/23 20:27
* @Author: hujiaming
* @Description:
 */

import (
	"flag"
	"fmt"
	"log"
	"net"
	"strconv"
	"time"
)

var (
	close = "close"
	how   = 0
)

func main() {
	flag.StringVar(&close, "close", close, "close or shutdown")
	flag.IntVar(&how, "how", how, "how shutdown")
	flag.Parse()
	l, err := net.Listen("tcp", ":2000")
	if err != nil {
		log.Fatal(err)
	}

	log.Println("listening... ")
	for {
		c, err := l.Accept()
		if err != nil {
			panic(err)
		}
		log.Println("get conn", c.RemoteAddr())
		go handle(c, close, how)
	}
}

func handle(c net.Conn, close string, how int) {
	go func() {
		b := make([]byte, 1024)
		for {
			_, err := c.Read(b)
			if err != nil {
				fmt.Println("read err", err)
				return
			}
			fmt.Println("read", string(b))
		}
	}()
	go func() {
		for i := 10000; ; i++ {
			_, err := c.Write([]byte(strconv.Itoa(i)))
			if err != nil {
				fmt.Println("write err", err)
				return
			}
			time.Sleep(time.Second)
			fmt.Println("write", i)
		}
	}()
	// 决定关闭连接的方式
	if close == "close" {
		if err := c.Close(); err != nil {
			panic(err)
		}
		log.Println("closed")
	} else {
		switch how {
		case 0:
			c.(*net.TCPConn).CloseRead() // syscall.Shutdown(fd, 0)
		case 1:
			c.(*net.TCPConn).CloseWrite() // syscall.Shutdown(fd, 1)
		case 2:
			c.(*net.TCPConn).Close() // syscall.Shutdown(fd, 2)
		}
		log.Println("shutdown, how:", how)
	}
}
```

### 7. 关于 Keep-Alive

TCP 也有自己的保活机制—— `Keep-Alive` 。它的工作机制是这样的：定义一个时间段，在这个时间段内，如果没有任何连接相关的活动，TCP 的保活机制就会启动，每隔一定时间，就向对侧发送一个数据量非常少 **探测报文**，如果连续几个探测报文都没有得到响应，那么就认为这个 TCP 连接已经失效，内核会将错误信息上报给上层应用。

相关的参数如下：

```bash
tcp_keepalive_time (integer; default: 7200; since Linux 2.2) 连接闲置 7200s 后
tcp_keepalive_intvl (integer; default: 75; since Linux 2.4) 每间隔 75s 发送一次探测报文
tcp_keepalive_probes (integer; default: 9; since Linux 2.2) 探测 9 次还未收到反馈就认为连接失效
```

开启了 TCP 的 `Keep-Alive` 之后，需要考虑下面三种情况：

1.  对端程序正常响应。则 TCP 保活时间被重置，等待下一个保活机制触发；
2.  对端程序崩溃。探测报文不可达，在经过 `75s * 9` 尝试之后，确认连接失效，断开连接；
3.  对端程序崩溃后重启。探测报文到达后，对端发现并不是当前连接的有效报文，于是回复一个 `RST` 报文，本端收到后会进行对应的重连操作。

TCP 的 `Keep-Alive` 选项默认关闭，而且在默认情况下，在 Linux 系统中，最少要经过 `2小时 11 分 15 秒`(上述三个时间的总和) 才能发现一条死亡连接，这似乎有点难以接受。

当然，我们可以手动改这个默认值：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import time
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

"""开启keepalive"""
s.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)
"""设置每20秒发送一次心跳包"""
s.setsockopt(socket.SOL_TCP, socket.TCP_KEEPIDLE, 20)
"""对方没有回应心跳包后，每隔一秒发送一次心跳包"""
s.setsockopt(socket.SOL_TCP, socket.TCP_KEEPINTVL, 1)
s.connect(('127.0.0.1', 8888)) 
time.sleep(200)
```

### 8. I/O 模型

如果拿去书店买书举例子。

阻塞 I/O 对应什么场景呢？ 你去了书店，告诉老板（内核）你想要某本书，然后你就一直在那里等着，直到书店老板翻箱倒柜找到你想要的书，有可能还要帮你联系全城其它分店。注意，这个过程中你一直滞留在书店等待老板的回复，好像在书店老板这里"阻塞"住了。

那么非阻塞 I/O 呢？你去了书店，问老板有没你心仪的那本书，老板查了下电脑，告诉你没有，你就悻悻离开了。一周以后，你又来这个书店，再问这个老板，老板一查，有了，于是你买了这本书。注意，这个过程中，你没有被阻塞，而是在不断轮询。

但轮询的效率太低了，于是你向老板提议：“老板，到货给我打电话吧，我再来付钱取书。”这就是前面讲到的 I/O 多路复用。

再进一步，你连去书店取书也想省了，得了，让老板代劳吧，你留下地址，付了书费，让老板到货时寄给你，你直接在家里拿到就可以看了。这就是异步I/O。

这几个 I/O 模型，再加上进程、线程模型，构成了整个网络编程的知识核心。

>    非阻塞 I/O 可以使用在 read、write、accept、connect 等多种不同的场景，在非阻塞
>   I/O 下，使用轮询的方式引起 CPU 占用率高，所以一般将非阻塞 I/O 和 I/O 多路复用技术
>   select、poll 等搭配使用，在非阻塞 I/O 事件发生时，再调用对应事件的处理函数。这种方
>   式，极大地提高了程序的健壮性和稳定性，是 Linux 下高性能网络编程的首选。

>   epoll 通过改进的接口设
>   计，避免了用户态 - 内核态频繁的数据拷贝，大大提高了系统性能。在使用 epoll 的时候，
>   我们一定要理解条件触发和边缘触发两种模式。条件触发的意思是只要满足事件的条件，比
>   如有数据需要读，就一直不断地把这个事件传递给用户；而边缘触发的意思是只有第一次满
>   足条件的时候才触发，之后就不会再传递同样的事件了。

## 三、`Golang` 实现

### 1. 链路层

主要讲解 **以太网网卡**、**虚拟网卡** 和 `ARP` 协议。

数据链路层属于计算机网络的底层，使用的信道主要有点对点信道和广播信道两种类型。 在 TCP/IP 协议族中，数据链路层主要有以下几个目的：

1.  接收和发送链路层数据，提供 io 的能力。
2.  为 IP 模块发送和接收数据
3.  为 ARP 模块发送 ARP 请求和接收 ARP 应答
4.  为 RARP 模块发送 RARP 请求和接收 RARP 应答

TCP/IP 支持多种不同的链路层协议，这取决于网络所使用的硬件。 数据链路层的协议数据单元—**帧**：将 IP 层（网络层）的数据报添加首部和尾部封装成帧。 数据链路层协议有许多种，都会解决三个基本问题：封装成帧，透明传输，差错检测。

**以太网协议**：是当今现有局域网采用的最通用的通信协议标准，故可认为以太网就是局域网。以太网实在应用太广了，以至于我们在现实生活中看到的链路层协议的数据封装都是以太网协议封装的。

**链路层寻址**：通信当然得知道发送者的地址和接受者的地址，这是最基础的。以太网规定，所有连入网络的设备，都必须具有“网卡”接口。然后数据包是从一块网卡，传输到另一块网卡的。网卡的地址，就是数据包的发送地址和接收地址，叫做 `MAC` 地址，也叫物理地址，这是最底层的地址。每块网卡出厂的时候，都有一个全世界独一无二的 MAC 地址，长度是 48 个二进制位，通常用 12 个十六进制数表示。有了这个地址，我们可以定位网卡和数据包的路径了。

**MTU** ：表示在链路层最大的传输单元，也就是链路层一帧数据的数据内容最大长度，单位为字节，MTU 是协议栈实现一个很重要的参数，请大家务必理解该参数。一般网卡默认 MTU 是 1500，当你往网卡写入的内容超过 1518bytes，就会报错，后面我们可以写代码试试。

## 四、参考资料

-   [UNIX网络编程 卷1：套接字联网API（第3版）](https://book.douban.com/subject/4859464/)
-    [Linux Socket编程（不限Linux）](https://www.cnblogs.com/skynet/archive/2010/12/12/1903949.html)
-   [揭开Socket编程的面纱](https://www.cnblogs.com/goodcandle/archive/2005/12/10/socket.html)
-   [https://github.com/ICKelin/gtun](https://github.com/ICKelin/gtun)
-   [https://github.com/bigeagle/gohop](https://github.com/bigeagle/gohop)
-   [https://docs.ioin.in/writeup/medium.com/__FWTO_O_gohop_vpn_E6_BA_90_E7_A0_81_E5_88_86_E6_9E_90_a131c70cf95c/index.html](https://docs.ioin.in/writeup/medium.com/__FWTO_O_gohop_vpn_E6_BA_90_E7_A0_81_E5_88_86_E6_9E_90_a131c70cf95c/index.html)
-   

## Notes

ARPANET 早期使用一种网络控制协议（Network Control Protocol，NCP）来达到主机与主机之间的通信，但是它无法和个别的计算机网络做交流，因为设备之间没有一个标准协议。1972 年，ARPANET 项目组的核心成员 Vinton Cerf 和 Bob Kahn 开始合作开展所谓的网络互联相互（Interneting Project）。他们希望连接不同的网络，使得一个网络上的主机能够与另一个主机网络上进行通信，需要克服的问题很多：不同的分组大小、不同的接口类型、不同的传输速率、以及不同的可靠性要求。Cerf 和 Kahn 提出利用被称为网关的一种设备作为中间的硬件，进行一个网络到另一个网络的数据传输。

之后 Cerf 和 Kahn 在 1974 年发表了里程碑式的文章 `Protocol for Packet Network Interconnection`，描述了实现端到端数据投递的协议，这是一个新版的 NCP，叫传输控制协议（TCP）。这篇文章包括了封装、数据报、网关的功能等概念，其中主要思想是把纠错功能从 IMP 移到了主机。同时该协议（TCP）被应用到 ARPANET 网络，但是此时依然没有形成一个网络标准，各种协议并存包括 NCP，TCP 等协议。

在 1997 年后，TCP 被拆分成两个网络协议：传输控制协议（TCP）和因特网协议（IP），IP 处理数据包的路由选择，TCP 负责高层次的功能，如分段、重组、检错。这个新的联合体就是人们熟知的 TCP/IP。

1980 年发表 UDP 协议。

1981 年 UNIX 系统集成了 TCP/IP 协议栈，包含网络软件的流行操作系统对网络的普及起了很大的作用。

1983 年原先的交流协议 NCP 被禁用，TCP/IP 协议变成了 ARPANET 的正式协议，同时 ARPANET 分裂成两个网络：军用网（MILNET）和非军用的 ARPANET。之后，NCP 成为历史，TCP/IP 开始成为通用协议。

1984 年 ISO 发布了开放式系统互联模型（OSI）。

再之后，互联网极速发展，更多的主干网被搭建，更多的主机连接进来，直至组成了世界互联的巨大网络。

**链路层**：链路层也是将数据包发送到另一台主机，但是这两台主机一定是同个局域网的（不考虑广域网二层打通的情况），链路层负责将网络层交下来的 IP 数据报组装成帧，在两个相邻节点间的链路上传送帧。链路层的通信就像在一栋小楼里面互相讲话一下，小明想与小红讲话，只要在楼里喊一下，“小红你在吗？”，小红听到了就会回复说，“小明，我在啊”。小明在喊小红的时候，在这栋楼里的其他人也听得到，这种行为叫**广播**。链路层网络不适合大型网络，因为一旦主机多了，广播会比较占用资源，就像楼里大家都在喊别人一下，听起来会很乱。

**一般的上网流程**：

![](https://doc.shiyanlou.com/document-uid949121labid10418timestamp1555395048260.png)

一般情况家里的上网流程如下，但不是一定是这样，请读者注意！

-   首先你得购买互联网服务提供商（ISP，如：中国电信）提供的账号密码；
-   启动家用路由器，假设路由器内网地址为 192.168.1.1，接着配置账号密码，通过拨号和 ISP 建立连接，ISP 会返回一个公网 IP 地址，假如 IP 为 1.1.10.1；
-   然后再把电脑插到家用路由器的网口上，那么电脑就获取到了内网 IP 地址，假如为 192.168.1.2，这时候家用路由器就是电脑的默认网关，和家用路由器的相连的网卡假设为 en0；
-   当在浏览器访问 `https://www.baidu.com` 时，浏览器会发起 DNS 请求得到对应的 IP，假如为 180.97.33.18，DNS 请求的详细过程我们暂时忽略；
-   拿到 IP 后，浏览器会使用 tcp 连接系统调用和远端主机建立连接，系统调用会进入内核；
-   内核先通过路由最长匹配查询目标 IP 下一跳地址，也就是邻居地址，比如目的 180.97.33.18 会匹配下一跳地址 192.168.1.1;
-   内核接着查询 ARP 表，得知下一跳地址的网卡和物理 MAC 地址，如果没有查询到，则会发送广播 ARP 请求，得到 MAC 地址；
-   到目前为止发送 tcp 报文所需的信息都有了，目标 IP 和目标 MAC 地址，此时系统会给 tcp 的连接分配一个源端口，假如为 33306；
-   之后完成 tcp 三次握手，将 HTTP 请求报文封装在 tcp 数据段中发送给网卡 en0；
-   家用路由器接收到电脑的数据报，经过源地址转换（SNAT），将数据报文发送给 ISP；
-   ISP 通过路由表选择发送给下一个路由，经过多个路由转发最终达到百度的服务主机；
-   百度服务器得到电脑发送的报文，返回 HTTP 响应，按原路返回给家用路由器；
-   家用路由器接收到 HTTP 响应报文后，经过目标地址转换（DNAT），将数据发送给电脑；
-   电脑上的浏览器接收到 HTTP 响应，渲染页面，呈现出网页；



## 极客时间《网络编程实战》摘抄

大家经常说的四层、七层，分别指的是什么？
TCP 三次握手是什么，TIME_WAIT 是怎么发生的？CLOSE_WAIT 又是什么状态？
Linux 下的 epoll 解决的是什么问题？如何使用 epoll 写出高性能的网络程序？
什么是网络事件驱动模型？Reactor 模式又是什么？



事实上，我认为学习高性能网络编程，掌握两个核心要点就可以了：第一就是理解网络协
议，并在这个基础上和操作系统内核配合，感知各种网络 I/O 事件；第二就是学会使用线
程处理并发。抓住这两个核心问题，也就抓住了高性能网络编程的“七寸”。



无论是客户端的 connect，还是服务端的 accept，或者 read/write 操
作等，socket 是我们用来建立连接，传输数据的唯一途径。

socket 是加州大学伯克利分校的研究人员在 20 世纪 80 年代早期提出的，所以也被叫做伯
克利套接字。伯克利的研究者们设想用 socket 的概念，屏蔽掉底层协议栈的差别。第一版
实现 socket 的就是 TCP/IP 协议，最早是在 BSD 4.2 Unix 内核上实现了 socket。很快大
家就发现这么一个概念带来了网络编程的便利，于是有更多人也接触到了 socket 的概念。
Linux 作为 Unix 系统的一个开源实现，很早就从头开发实现了 TCP/IP 协议，伴随着
socket 的成功，Windows 也引入了 socket 的概念。于是在今天的世界里，socket 成为
网络互联互通的标准。



之所以对 UDP 使用 connect，绑定本地地址和端口，是为了让我们的程序可以快速获取异步错误信息的通知，同时也可以获得一定性能上的提升。

在所有 TCP 服务器程序中，调用 bind 之前请设置 SO_REUSEADDR 套接字选项，以便服务端程序可以在极短时间内复用同一个端口启动。



ep_poll_callback 函数的作用非常重要，它将内核事件真正地和 epoll 对象联系了起来。它
又是怎么实现的呢？
首先，通过这个文件的 wait_queue_entry_t 对象找到对应的 epitem 对象，因为
eppoll_entry 对象里保存了 wait_quue_entry_t，根据 wait_quue_entry_t 这个对象的地
址就可以简单计算出 eppoll_entry 对象的地址，从而可以获得 epitem 对象的地址。这部
分工作在 ep_item_from_wait 函数中完成。一旦获得 epitem 对象，就可以寻迹找到
eventpoll 实例。