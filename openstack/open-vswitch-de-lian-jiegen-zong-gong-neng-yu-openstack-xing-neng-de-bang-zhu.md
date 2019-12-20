**概述**

**  
**

防火墙是避免IT设备在多变的环境中受到安全攻击的必要设施。而有效、有意义的防火墙大多需要实时跟踪来往于不同设备的各类连接，即需为“状态防火墙（stateful firewall）”，因为即使是“仅允许本机与互联网上的服务器的单向连接而拒绝自互联网的设备连接本机”这样的基本规则也需要由状态防火墙实现。对于虚拟设备亦是如此。显然，任何可用的云平台同样需要此类保护。

  


**基于OpenStack的状态防火墙**

**  
**

OpenStack为其访客VM实施的有状态防火墙是OpenStack Security Group Feature的核心–管理程序可由此避免VM受到无价值数据流的影响。

  


状态防火墙中的主机（OpenStack节点）需实时跟踪各个独立的连接，即完成 “conntrack”。需注意，“连接”与 “流”的定义不同 – 前者为双方向且需被建立，后者不涉及方向的概念，且无状态。

  


而Open vSwitch \(http://openvswitch.org/\) 则是被Neutron 用来实现OpenStack网络 - 连接VM并创建不同节点间的叠加网的可编程软件开关（Open vSwitch不是唯一一个可用的后端，但因其灵活性及社区地位，它的功能最多、性能最佳）。

  


但是Open vSwitch数据路径的数据包交换只能全部基于流，且无状态。这对于对状态防火墙的需求的确是个棘手问题。

  


**解决办法：调整iptables**

**  
**

此题并非无解。Linux内核设有连接跟踪模块，可用于实施状态防火墙。但这些功能仅限于IP协议层的Linux内核防火墙（即 “iptables”）。但是Open vSwitch并不能在IP协议层（又称L3）运行，而只能运行于其下一层（又称L2）。换言之，并非所有被内核处理的数据包都可以被iptables处理 – 只有被导向本地主机，或者由主机选择路径的数据包才可以。被交换的数据包（无论通过Linux网桥还是Open vSwitch）都不会被iptables处理。

  


而OpenStack同样需要VM处于L2段，即二者之间的数据包亦需要交换。但一些技巧仍然能够让OpenStack利用iptables实施状态防火墙。



Linux网桥（Linux内核中的传统软件交换机制）自带名为ebtables的过滤机制。虽然连接跟踪功能无法通过ebtables使用，合适的系统配置参数将可以从ebtables呼叫到iptables。这样，L2的数据包交换就也能用到连接跟踪了。

  


**那么该把它放到OpenStack数据包路径里的哪个部分呢？**

  


OpenStack节点的核心是 “整合网桥（integration bridge）”,br-int。一般部署中，br-int通过Open vSwitch实施，它负责在VM之间、节点之间指挥数据包的去向。如此，每个VM都与一个整合网桥相连。

  


而状态防火墙需要位于VM与整合网桥之间。这就意味着利用iptables需要在VM与整合网桥之间设Linux网桥。该网桥需要有正确的设置以能够呼叫iptables；且iptables规则需要启用conntrack及必要的防火墙。

  




**示例**

**  
**

![](https://mmbiz.qpic.cn/mmbiz_png/TkGS5WgibziaribH9ZbJwiayXQ3pR1wBrIxt1Ky72YlOjYicicE4MAeYLAOxbTdq0EA5PEEWwU80cRQa6TG0r5QcFwIQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


  


数据包从VM到网络堆栈的路径示意图



第一个VM通过tap1接口与主机相连。之后，从VM输出的数据包被导向Linux网桥qbr1。在那里，ebtables将呼叫iptalbes，而进向的数据包也将根据配置好的规则匹配。完成匹配的数据包将通过网桥，被导向与网桥相连的第二个接口，也就是qvb1，veth 对组之一。

  


Veth对组是一对在内部互连的接口。被对组的任意一方发送的内容将被另一方接收，反之亦然。Veth对组的必要性在于，Linux网桥与Open vSwitch整合网桥之间需设互连装置。

  


之后，数据包到达br-int并被导向第二个VM，在离开br-int后前往qvo2，再到达网桥qbr2。该数据包先后通过ebtables与iptables，最终到达目标VM tap2。

  


这是个复杂的过程– 这些网桥与接口会为CPU处理速度和延时产生附加成本，影响性能。

  


**基于Open vSwitch的连接跟踪**

**  
**

上述过程可被大幅简化 - 把连接跟踪直接包含进Open vSwitch即可。

  


近期，Linux内核中定义连接跟踪的代码已与iptables分离，Open vSwitch也得到了conntrack的支持。这让匹配连接（而不仅是流）成为可执行的方案 Jakub Libosvar \(Red Hat\) 在Neutron中启用了这个新功能。

  


目前，VM已可以直接与整合网桥连接，状态防火墙亦可通过Open vSwitch规则被独立实施。

  


![](https://mmbiz.qpic.cn/mmbiz_png/TkGS5WgibziaribH9ZbJwiayXQ3pR1wBrIxtw1RIjGNBz2Y9icRAbjjibTTibGO5FBPW4S5cDXLrF5d4h0FOzzlGQJhXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

改进之后的流程的示意图

  


来自第一个VM的数据包被直接导向br-int，由预先配置好的规则检查，之后或被直接拒绝，或被接受并输出至第二个VM（tap2）。



这个流程可大幅降低数据包的处理成本，有效提升性能。以下为省去的资源与成本支出：

  


1. **veth对组中数据包的入队**：送往veth终端的数据包可被入队或退队，适时进行处理。

  

2. **按VM网桥计算的网桥成本**：所有通过网桥的数据包都需经过FDB （Forwarding Database）处理。

  

3. **ebtables 的日常成本**：经计算，仅启用ebtables而不加设任何配置规则亦可对网桥输出能力产生损耗。ebtables已被普遍认为是过时的技术，也已基本被停止维护了，特别是性能方面。

  

4. **iptables的日常成本**：目前对于iptables还没有按接口执行配置规则的概念 – iptables的规则是通用的，即对于每个数据包，其进入接口都需被核对，其规则执行被分发至与该接口相符合的规则。这意味着必需使用接口名称进行线性搜索，一个高成本手段，特别在VM数量较高时。

  


相比之下，使用vSwitch conntrack可直接避免第一到三点。Open vSwitch只涉及通用规则，所以进入接口的匹配仍是必要的。但不同于iptables，该匹配通过part number\(而非文本类的接口名称\)完成。更重要的是，通过使用hash table，第四点也被完全避免了。

  


之后，防火墙自身的规则成为仅剩的日常成本。

  


**综上所述**

**  
**

没有Open vSwitch conntrack时，

  


* Linux网桥需位于VM与整合网桥之间；

* 该网桥需通过一个veth对组连接至整合网桥；

* 需要通过网桥的数据包由ebtables和iptables处理，并将实施状态防火墙；

* veth、网桥、ebtables和iptables的日常成本均较高。

  


启用 Open vSwitch conntrack后:

* VM直接与整合网桥连接；

* 状态防火墙可在整合网桥，通过hash tables 被直接实施



