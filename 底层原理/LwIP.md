## 参考资料

-   [https://blog.csdn.net/bytxl/article/details/50318745](https://blog.csdn.net/bytxl/article/details/50318745)
-   [嵌入式网络那些事：LwIP 协议深度详解与 实战演练 第二版](http://120.27.32.195/uploads/bookfile/201507/9787517033127_1.pdf)

![](https://img-blog.csdn.net/20151215180214839)

-   理解为什么 `ARP` 协议是非常重要的？

`ARP` 协议的作用是 **实现 `以太网MAC` 地址到 `IP` 地址的映射**。以太网底层的数据发送依赖于网卡的物理地址(即 MAC 地址)，而不是主机的 IP，通过 `ARP` 协议，主机可以得到相邻结点的 IP 地址和 MAC 地址，为以太网的数据包交换提供保证。