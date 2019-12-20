**bond**

![](https://mmbiz.qpic.cn/mmbiz_png/THOMb1XdkLNt70b8Nia80KNUvAzLkYKJx1iaSC5ZECplnPFJzOfYhVvI8nRcXfThebPR9KoMRxnMtPzj4tF4coTA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在企业及电信Linux服务器环境上，网络配置都会使用Bonding技术做网口硬件层面的冗余，防止单个网口应用的单点故障。Bond有两种典型的模式：主备，负载均衡。无论哪种模式，Bonding技术都是用来实现网口故障后平滑切换的。

  


**一**

**主备模式**

  


主备模式下，Bonding实现会将Bond的两个slave网口的MAC地址改为Bond的MAC地址，而Bond的MAC地址是Bond创建启动后，主用slave网口的MAC地址一样的。当主用网口故障后，Bond会切换到备用网口，切换过程中，上层的应用是无感知不受影响的。

  


**二**

**负载均衡模式**

  


负载均衡模式下， Bonding实现可以保持两个slave网口的MAC地址不变，Bond的MAC地址是其中一个网卡的，Bond MAC地址的选择是根据Bond自己实现的一个算法来的。

  


**LACP**

![](https://mmbiz.qpic.cn/mmbiz_png/THOMb1XdkLNt70b8Nia80KNUvAzLkYKJx1iaSC5ZECplnPFJzOfYhVvI8nRcXfThebPR9KoMRxnMtPzj4tF4coTA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

LACP，基于IEEE802.3ax标准的LACP（Link Aggregation Control Protocol，链路汇聚控制协议）是一种实现链路动态汇聚的协议。LACP协议通过LACPDU（Link Aggregation Control Protocol Data Unit，链路汇聚控制协议数据单元）与对端交互信息。

  


启用某端口的LACP协议后，该端口将通过发送LACPDU向对端通告自己的系统优先级、系统MAC地址、端口优先级、端口号和操作Key。对端接收到这些信息后，将这些信息与其它端口所保存的信息比较以选择能够汇聚的端口，从而双方可以对端口加入或退出某个动态汇聚组达成一致。

  


端口汇聚是将多个端口汇聚在一起形成一个汇聚组，以实现出/入负荷在汇聚组中各个成员端口中的分担，同时也提供了更高的连接可靠性。

  


**一**

**静态汇聚**

  


**1、静态lacp汇聚由用户进行手工配置，不允许系统自动添加或删除汇聚组中的端口。**

  


静态汇聚端口的lacp协议为激活状态，当一个静态汇聚组被删除时，其成员端口将形成一个或多个动态lacp汇聚，并保持lacp的被激活。禁止用户关闭静态汇聚端口的lacp协议。

  


**2、 静态汇聚组中的端口状态**

  


在静态汇聚组中，端口可能处于两种状态：selected或standby。selected端口和standby端口都能收发lacp协议，但standby端口不能转发用户报文。

  


**1）**

在静态汇聚组中，系统按照以下原则设置端口处于selected或者standby状态：系统按照端口全双工/高速率、全双工/低速率、半双工/高速率、半双工/低速率的优先次序，选择优先次序最高的端口处于selected状态，其他端口则处于standby状态。

**2）**

和 selected状态的最小端口所连接的对端设备不同的端口处于standby状态。

**3）**

和selected状态的最小端口所连接的对端设备相同，但端口在不同的汇聚组内的端口将处于standby状态。

  


**4）**

端口因存在硬件限制（如不能跨板汇聚）无法汇聚在一起，而无法与处于selected状态的最小端口汇聚的端口将处于standby状态。

  


**5）**

与处于selected状态的最小端口的基本配置不同的端口将处于standby状态。

  


由于设备所能支持的汇聚组中的selected端口数有限制，如果当前的成员端口数超过了设备所能支持的最大selected端口数，系统将按照端口号从小到大的顺序选择一些端口为selected端口，其他则为standby端口。

  


二

**LACP动态汇聚**

  


**1、动态lacp汇聚概述**

  


动态lacp汇聚是一种系统自动创建/删除的汇聚，允许用户增加或删除动态lacp汇聚中的成员端口。只有速率和双工属性相同、连接到同一个设备、有相同基本配置的端口才能被动态汇聚在一起。即使只有一个端口也可以创建动态汇聚，此时为单端口汇聚。动态汇聚中，端口的lacp协议处于使能状态。

  


**2、动态汇聚组中的端口状态**

  


在动态汇聚组中，端口可能处于两种状态：selected或standby。selected端口和standby端口都能收发lacp协议，但standby端口不能转发用户报文。由于设备所能支持的汇聚组中的最大端口数有限制，如果当前的成员端口数量超过了最大端口数的限制，则本端系统和对端系统会进行协商，根据设备id优的一端的端口id的大小，来决定端口的状态。具体协商步骤如下：

  


**1）**

比较设备id（系统优先级+系统mac地址）。先比较系统优先级，如果相同再比较系统mac地址。设备id小的一端被认为优。

  


**2）**

比较端口id（端口优先级+端口号）。对于设备id优的一端的各个端口，首先比较端口优先级，如果优先级相同再比较端口号。端口id小的端口为selected端口，剩余端口为standby端口。

  


**3）**

在一个汇聚组中，处于selected状态且端口号最小的端口为汇聚组的主端口，其他处于selected状态的端口为汇聚组的成员端口。

  


**三**

**LACP工作模式**

启动LACP的端口可以有两种工作模式，passive和active。

  


**passive：**

被动模式，该模式下端口不会主动发送LACPDU报文，在接收到对端发送的LACP报文后，该端口进入协议计算状态。

  


**Active：**

主动模式，该模式下端口会主动向对端发送LACPDU报文，进行LACP协议的计算。

  


**虚拟交换机支持的bond模式**

![](https://mmbiz.qpic.cn/mmbiz_png/THOMb1XdkLNt70b8Nia80KNUvAzLkYKJxfmtYt3tst6JtibdeMKfxwJykwHRgcD02RYglgeorkfCVDQlzqodTUmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

虚拟交换机 bond支持以下几种模式，根据实际组网需求进行选择：

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/THOMb1XdkLNt70b8Nia80KNUvAzLkYKJxgPrLJmVd3XdxzBdl9WCpR1cgNFwNeekLDoibAvbQI9eFtY8DibrUBG0w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


其中3种模式的差别说明如下：

  


**1、**

**active-backup：**

主备模式

  


**2、**

**balance-slb：**

负荷分担，根据源MAC地址负荷分担

  


**3、**

**balance-tcp：**

负荷分担，根据IP地址+TCP端口进行负荷分担。

  


例如：网络类型为vlan，虚拟交换机配置bond模式为balance-tcp，则虚拟交换机需要同时设置lacp工作在active模式。此时，对端的物理交换机需要设置bond，模式选择与虚拟交换机侧相同的balance-tcp，lacp工作方式可以选择active或者passive。

  


**linux支持的bond模式**

![](https://mmbiz.qpic.cn/mmbiz_png/THOMb1XdkLNt70b8Nia80KNUvAzLkYKJxfmtYt3tst6JtibdeMKfxwJykwHRgcD02RYglgeorkfCVDQlzqodTUmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_jpg/THOMb1XdkLNt70b8Nia80KNUvAzLkYKJxgZChyn5XL7kib0vqrEBSkvVCWoPppY9ByHiaSpQEhbbkDGRKZ6cl2MzA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


  


**模式0：**

mode=0,被称为平衡轮循策略：特点：传输数据包顺序是依次传输

  


**模式1：**

mode=1,被称为主备策略

  


**模式2：**

mode=2,被称为平衡策略特点：基于指定的传输HASH策略传输数据包

  


**模式3：**

mode=3,被称为广播策略特点：在每个slave接口上传输每个数据包

  


**模式4：**

mode=4,被称为LACP动态链路聚合特点：见LACP

  


**模式5：**

 mode=5,被称为传输负载均衡 不需要任何特别的switch\(交换机\)支持的通道bonding。在每个slave上根据当前的负载（根据速度计算）分配外出流量。如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址。

  


**模式6：**

mode=6,被 称为适应性负载均衡  特点：该模式包含了balance-tlb模式，同时加上针对IPV4流量的接收负载均衡\(receive load balance, rlb\)，而且不需要任何switch\(交换机\)的支持。接收负载均衡是通过ARP协商实现的。bonding驱动截获本机发送的ARP应答，并把源硬件地址改写为bond中某个slave的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。

