**1 概述**

**1.1 产生背景**

随着企业业务的快速扩展，IT作为基础设施，其快速部署和高利用率成为主要需求。云计算可以为之提供可用的、便捷的、按需的资源，成为当前企业IT建设的常规形态，而在云计算中大量采用和部署的虚拟化几乎成为一个基本的技术模式。部署虚拟机需要在网络中无限制地迁移到目的物理位置，虚拟机增长的快速性以及虚拟机迁移成为一个常态性业务。传统的网络已经不能很好满足企业的这种需求，面临着如下挑战：

  


**虚拟机迁移范围受到网络架构限制**

虚拟机迁移的网络属性要求，当其从一个物理机上迁移到另一个物理机上，虚拟机需要不间断业务，因而需要其IP地址、MAC地址等参数维持不变，如此则要求业务网络是一个二层网络，且要求网络本身具备多路径多链路的冗余和可靠性。传统的网络生成树（STP，Spaning Tree Protocol）技术不仅部署繁琐，且协议复杂，网络规模不宜过大，限制了虚拟化的网络扩展性。基于各厂家私有的IRF/vPC等设备级的（网络N:1）虚拟化技术，虽然可以简化拓扑、具备高可靠性，但是对于网络有强制的拓扑形状要求，在网络的规模和灵活性上有所欠缺，只适合小规模网络构建，且一般适用于数据中心内部网络。

  


**虚拟机规模受网络规格限制**

在大二层网络环境下，数据流均需要通过明确的网络寻址以保证准确到达目的地，因此网络设备的二层地址表项大小（即MAC地址表），成为决定了云计算环境下虚拟机的规模上限，并且因为表项并非百分之百的有效性，使得可用的虚拟机数量进一步降低。特别是对于低成本的接入设备而言，因其表项一般规格较小，限制了整个云计算数据中心的虚拟机数量，但如果其地址表项设计为与核心或网关设备在同一档次，则会提升网络建设成本。虽然核心或网关设备的MAC与ARP规格会随着虚拟机增长也面临挑战，但对于此层次设备能力而言，大规格是不可避免的业务支撑要求。减小接入设备规格压力的做法可以是分离网关能力，如采用多个网关来分担虚拟机的终结和承载，但如此也会带来成本的巨幅上升。

  


**网络隔离/分离能力限制**

当前的主流网络隔离技术为VLAN（或VPN），在大规模虚拟化环境部署会有两大限制：一是VLAN数量在标准定义中只有12个比特单位，即可用的数量为4K，这样的数量级对于公有云或大型虚拟化云计算应用而言微不足道，其网络隔离与分离要求轻而易举会突破4K；二是VLAN技术当前为静态配置型技术，这样使得整个数据中心的网络几乎为所有VLAN被允许通过（核心设备更是如此），导致任何一个VLAN的未知目的广播数据会在整网泛滥，无节制消耗网络交换能力与带宽。

  


上述的三大挑战，完全依赖于物理网络设备本身的技术改良，目前看来并不能完全解决大规模云计算环境下的问题，一定程度上还需要更大范围的技术革新来消除这些限制，以满足云计算虚拟化的网络能力需求。在此驱动力基础上，逐步演化出Overlay网络技术。

  


**1.2 技术优点**

Overlay是一种叠加虚拟化技术，主要具有以下优点：

  


基于IP网络构建Fabric。无特殊拓扑限制，IP可达即可；承载网络和业务网络分离；对现有网络改动较小，保护用户现有投资。

16M多租户共享，极大扩展了隔离数量。

网络简化、安全。虚拟网络支持L2、L3等，无需运行LAN协议，骨干网络无需大量VLAN Trunk。

支持多样化的组网部署方式，支持跨域互访。

支持虚拟机灵活迁移，安全策略动态跟随。

转发优化和表项容量增大。消除了MAC表项学习泛滥，ARP等泛洪流量可达范围可控，且东西向流量无需经过网关。

  


**2 Overlay技术介绍**

**2.1 Overlay的概念介绍**

在网络技术领域，Overlay是一种网络架构上叠加的虚拟化技术模式，其大体框架是对基础网络不进行大规模修改的条件下，实现应用在网络上的承载，并能与其它网络业务分离，并且以基于IP的基础网络技术为主（如图1所示）。

1）Overlay网络是指建立在已有网络上的虚拟网，由逻辑节点和逻辑链路构成。  
2）Overlay网络具有独立的控制和转发平面，对于连接在overlay边缘设备之外的终端系统来说，物理网络是透明的。

3）Overlay网络是物理网络向云和虚拟化的深度延伸，使云资源池化能力可以摆脱物理网络的重重限制，是实现云网融合的关键。

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavRicAkceE33L8ZWBZH2MXoQMJdhMoHfuK7kBGaQ0ia2ibUrDIsZHwMtFhA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
               图1 Overlay网络概念图  
  


**2.2 Overlay的解决方法**

针对前文提到的三大挑战，Overlay给出了完美的解决方法。

  


**针对虚拟机迁移范围受到网络架构限制的解决方式**

Overlay把二层报文封装在IP报文之上，因此，只要网络支持IP路由可达就可以部署Overlay网络，而IP路由网络本身已经非常成熟，且在网络结构上没有特殊要求。而且路由网络本身具备良好的扩展能力，很强的故障自愈能力和负载均衡能力。采用Overlay技术后，企业不用改变现有网络架构即可用于支撑新的云计算业务，极方便用户部署。

  


**针对虚拟机规模受网络规格限制的解决方式**

虚拟机数据封装在IP数据包中后，对网络只表现为封装后的网络参数，即隧道端点的地址，因此，对于承载网络（特别是接入交换机），MAC地址规格需求极大降低，最低规格也就是几十个（每个端口一台物理服务器的隧道端点MAC）。当然，对于核心/网关处的设备表项（MAC/ARP）要求依然极高，当前的解决方案仍然是采用分散方式，通过多个核心/网关设备来分散表项的处理压力。

  


**针对网络隔离/分离能力限制的解决方式**

针对VLAN只能支持数量4K以内的限制，在Overlay技术中扩展了隔离标识的位数，可以支持高达16M的用户，极大扩展了隔离数量。

  


**3 Overlay技术实现**

**3.1 Overlay网络基础架构**

VXLAN（Virtual eXtensible LAN，可扩展虚拟局域网络）是基于IP网络、采用“MAC in UDP”封装形式的二层VPN技术，具体封装的报文格式如图2所示。VXLAN可以基于已有的服务提供商或企业IP网络，为分散的物理站点提供二层互联功能，主要应用于数据中心网络。

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavW4GEIUnI6O4cmIZDuqUbONMW0YFJWXqxIKWg9n5OtlCe5NhXEyfZpw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
图2 VXLAN报文格式  
  


VXLAN技术已经成为目前Overlay技术事实上的标准，得到了非常广泛的应用。

  


以VXLAN技术为基础的Overlay网络架构模型如图3所示：  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavyp6wwKpsS6KsWvHTpVN8Z5V2NVMBaW0T1Qddiat1bWTsKaPIfYGRB8w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
  
                      图3 Overlay网络的基础架构  
  
  


**VM（Virtual Machine，虚拟机）**

在一台服务器上可以创建多台虚拟机，不同的虚拟机可以属于不同的VXLAN。属于相同VXLAN的虚拟机处于同一个逻辑二层网络，彼此之间二层互通。

  


两个VXLAN可以具有相同的MAC地址，但在一个VXLAN范围段内不能有一个重复的MAC地址。

  


**VTEP（VXLAN Tunnel End Point，VXLAN隧道端点）**

VXLAN的边缘设备，进行VXLAN业务处理：识别以太网数据帧所属的VXLAN、基于VXLAN对数据帧进行二层转发、封装/解封装VXLAN报文等。

  


VXLAN通过在物理网络的边缘设置智能实体VTEP，实现了虚拟网络和物理网络的隔离。VTEP之间建立隧道，在物理网络上传输虚拟网络的数据帧，物理网络不感知虚拟网络。VTEP将从虚拟机发出/接受的帧封装/解封装，而虚拟机并不区分VNI和VXLAN隧道。

  


**VNI（VXLAN Network Identifier，VXLAN网络标识符）**

VXLAN采用24比特标识二层网络分段，使用VNI来标识二层网络分段，每个VNI标识一个VXLAN，类似于VLAN ID作用。VNI占用24比特，这就提供了近16M可以使用的VXLAN。VNI将内部的帧封装（帧起源在虚拟机）。使用VNI封装有助于VXLAN建立隧道，该隧道在第三层网络之上覆盖第二层网络。

  


**VXLAN隧道**

在两个VTEP之间完成VXLAN封装报文传输的逻辑隧道。业务报文在入隧道时进行VXLAN头、UDP头、IP头封装后，通过三层转发透明地将封装后的报文转发给远端VTEP，远端VTEP对其进行出隧道解封装处理。

  


**VSI（Virtual Switching Instance，虚拟交换实例）**

VTEP上为一个VXLAN提供二层交换服务的虚拟交换实例。

  


**3.2 Overlay网络部署需求**

**3.2.1 VXLAN网络和传统网络互通的需求**

为了实现VLAN和VXLAN之间互通，VXLAN定义了VXLAN网关。VXLAN网关上同时存在VXLAN端口和普通端口两种类型端口，它可以把VXLAN网络和外部网络进行桥接、完成VXLAN ID和VLAN ID之间的映射和路由。和VLAN一样，VXLAN网络之间的通信也需要三层设备的支持，即VXLAN路由的支持。同样VXLAN网关可由硬件设备和软件设备来实现。

  


当收到从VXLAN网络到普通网络的数据时，VXLAN网关去掉外层包头，根据内层的原始帧头转发到普通端口上；当有数据从普通网络进入到VXLAN网络时，VXLAN网关负责打上外层包头，并根据原始VLAN ID对应到一个VNI，同时去掉内层包头的VLAN ID信息。相应的如果VXLAN网关发现一个VXLAN包的内层帧头上还带有原始的二层VLAN ID，会直接将这个包丢弃。

  


如图4左侧所示，VXLAN网关最简单的实现应该是一个Bridge设备，仅仅完成VXLAN到VLAN的转换，包含VXLAN到VLAN的1：1、N：1转换，复杂的实现可以包含VXLAN Mapping功能实现跨VXLAN转发，实体形态可以是vSwitch、物理交换机。

  


如图4右侧所示，VXLAN路由器（也称为VXLAN IP GW）最简单的实现可以是一个Switch设备，支持类似VLAN Mapping的功能，实现VXLAN ID之间的Mapping，复杂的实现可以是一个Router设备，支持跨VXLAN转发，实体形态可以是NFV形态的路由器、物理交换机、物理路由器。

  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavqLbQXSmK0Cz1gHKWb5bIibqocOmNXmuTFEiaZaibgb6GicJM0amtInU5DQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
              图4 VXLAN网关和VXLAN路由简单实现  


**3.2.2 VXLAN网络安全需求**

同传统网络一样，VXLAN网络同样需要进行安全防护。

VXLAN网络的安全资源部署需要考虑两个需求：

  


**VXLAN和VLAN之间互通的安全控制**

传统网络和Overlay网络中存在流量互通，需要对进出互通的网络流量进行安全控制，防止网络间的安全问题。针对这种情况，可以在网络互通的位置部署VXLAN防火墙等安全资源，VXLAN防火墙可以兼具VXLAN网关和VXLAN路由器的功能，该功能可以称之为南北向流量安全。

  


**VXLAN ID对应的不同VXLAN域之间互通的安全控制**

VM之间的横向流量安全是在虚拟化环境下产生的特有问题，在这种情况下，同一个服务器的不同VM之间的流量可能直接在服务器内部实现交换，导致外部安全资源失效。针对这种情况，可以考虑使用重定向的引流方法进行防护，又或者直接基于虚拟机进行防护，这个功能可以称之为东西向流量安全。

  


网络部署中的安全资源可以是硬件安全资源，也可以是软件安全资源，还可以是虚拟化的安全资源。  
  


**3.2.3 Overlay网络虚拟机位置无关性**  


通过使用MAC-in-UDP封装技术，VXLAN为虚拟机提供了位置无关的二层抽象，Underlay网络和Overlay网络解耦合。终端能看到的只是虚拟的二层连接关系，完全意识不到物理网络限制。

  


更重要的是，这种技术支持跨传统网络边界的虚拟化，由此支持虚拟机可以自由迁移，甚至可以跨越不同地理位置数据中心进行迁移。如此以来，可以支持虚拟机随时随地接入，不受实际所在物理位置的限制。

  


所以VXLAN的位置无关性，不仅使得业务可在任意位置灵活部署，缓解了服务器虚拟化后相关的网络扩展问题，而且使得虚拟机可以随时随地接入、迁移，是网络资源池化的最佳解决方式，可以有力地支持云业务、大数据、虚拟化的迅猛发展。

  


**3.2.4 Overlay与SDN的结合**

Overlay技术与SDN可以说天生就是适合互相结合的技术组合。前面谈到的Overlay网络虚拟机物理位置无关特性就需要有一种强有力的集中控制技术进行虚拟机的管理和控制。而SDN技术恰好可以完美的做到这一点。接下来就让我们继续分析Overlay技术和SDN技术相结合带来的应用场景。

  


**4 H3C SDN Overlay模型设计**

**4.1 H3C SDN Overlay模型设计**

在数据中心虚拟化多租户环境中部署和配置网络设施是一项复杂的工作，不同租户的网络需求存在差异，且网络租户是虚拟化存在，和物理计算资源位置无固定对应关系。通过传统手段部署物理网络设备为虚拟租户提供网络服务，一方面可能限制租户虚拟计算资源的灵活部署，另一方面需要网络管理员执行远超传统网络复杂度的网络规划和繁重的网络管理操作。在这种情况下，VPC（Virtual Private Cloud，虚拟私有云）技术就应运而生了。VPC对于网络层面，就是对物理网络进行逻辑抽象，构架弹性可扩展的多租户虚拟私有网络，对于私有云、公有云和混合云同样适用。

  


H3C的SDN控制器称为VCF控制器。H3C通过VCF控制器控制Overlay网络从而将虚拟网络承载在数据中心传统物理网络之上，并向用户提供虚拟网络的按需分配，允许用户像定义传统L2/L3网络那样定义自己的虚拟网络，一旦虚拟网络完成定义，VCF控制器会将此逻辑虚拟网络通过Overlay技术映射到物理网络并自动分配网络资源。VCF的虚拟网络抽象不但隐藏了底层物理网络部署的复杂性，而且能够更好的管理网络资源，最大程度减少了网络部署耗时和配置错误。

  


VCF控制器将虚拟网络元素组织为“资源池”，VCF控制器控制了“网络资源池”的按需分配，进而实现虚拟网络和物理网络的Overlay映射。

  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiaviblPFl8Kn5XvK6A9NhM1YcW7thOmMFEWK9mwl8kpbggRet58lf1tVyw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
图5 VPC多租户资源池场景  
VCF控制器的虚拟网络元素的抽象方式与OpenStack网络模型兼容，如图6所示：  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavfzXAqRSNLBunO9gLWeLWic9fZicLfoeGx5FCxGEMXKxxJymYPsU6q57Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
                 图6 VPC多租户资源池场景  
  
虚拟网络的各个要素如下表：  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiav4nyGiaYggYNfMf1dMWUwXkiafgdBoCfW38wFQWSXuQibFoENTLuNmAKew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
  
**4.2 SDN控制器模型介绍  
  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavicTMfnXLrr3oeWeJcgdXiarRashAVkTFLCkeYiaZ2gHFoqO7quib1YSkzw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**                       图7 SDN控制器模型  
  


从控制器是否参与转发设备的转发控制来看，当前主要有两种控制器类型：

  


**控制器弱控制模式**

弱控制模式下，控制平面基于网络设备自学习，控制器不在转发平面，仅负责配置下发，实现自动部署。主要解决网络虚拟化，提供适应应用的虚拟网络。

弱控制模式的优点是转发控制面下移，减轻和减少对控制器的依赖。

  


**控制器强控制模式**

在强控制模式下，控制器负责整个网络的集中控制，体现SDN集中管理的优势。

  


基于OpenFlow的强控制使得网络具备更多的灵活性和可编程性。除了能够给用户提供适合应用需要的网络，还可以集成FW等提供安全方案；可以支持混合Overlay模型，通过控制器同步主机和拓扑信息，将各种异构的转发模型同一处理；可以提供基于OpenFlow的服务链功能对安全服务进行编排，可以提供更为灵活的网络诊断手段，如虚拟机仿真和雷达探测等。

  


用户可能会担心强控制模式下控制器全部故障对网络转发功能的影响，这个影响因素可以通过下述两点来降低和消除：

1）通过控制器集群增加控制器可靠性，避免单点故障  


2）逃生机制：设备与所有控制器失联后，切换为自转模式，业务不受影响。

3）考虑到强控制模式可以支持混合Overlay模型，可以额外支持安全、服务链等灵活、可编程的功能，并且可靠性又可以通过上述方式加强，我们建议使用强控制模式来实现SDN Overlay。

  
**4.3 H3C SDN Overlay组件介绍  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavLE5ETwWicZ4qvoH8tsng3Y1pmBhPpUJiaiayt2w1lOFobgk8Ru3VWMOcw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**图8 H3C SDN Overlay组件  
  


如图8所示，H3C SDN Overlay主要包含如下组件：

  


**云管理系统**

可选，负责计算，存储管理的云平台系统，目前主要包括Openstack，Vmware Vcenter和H3C Cloud OS。

  


**VCF Controller集群**

必选，VCF Controller实现对于VPC网络的总体控制。

  


**VNF Manager**

VNF Manager实现对NFV设备（如vFW、vLB）的生命周期管理。

  


**VXLAN GW**

必选，VXLAN GW包括vSwitch、S68、VSR等，实现虚拟机、服务器等各种终端接入到VXLAN网络中。

  


**VXLAN IP GW**

必选，VXLAN IP GW包括S125-X、S98、VSR等，实现VXLAN网络和经典网络之间的互通。

  


**虚拟化平台**  


可选，vSwitch和VM运行的Hypervisor平台，目前主要包括CAS、VMware、KVM等。

  


**Service安全设备**

可选，包括VSR、vFW、vLB和M9000、安全插卡等设备，实现东西向和南北向服务链服务节点的功能。  
  
**4.4 SDN Overlay网络与云对接**

公有云或私有云（VPC）对网络的核心需求：

1）租户隔离  


2）网络自定义

3）资源大范围灵活调度

4）应用与网络位置无关

5）网络资源池化与按需分配

6）业务自动化

  


H3C提出的解决方案：

1）利用VXLAN Overlay提供一个“大二层”网络环境，满足资源灵活调度的需求；  


2）由SDN控制器VCFC实现对整个Overlay网络的管理和控制；

3）由VXLAN GW实现服务器到VXLAN网络的接入；

4）由VXLAN IP GW实现VXLAN网络与传统网络的对接；

5）NFV设备（vSR/vFW/vLB）实现东西向和南北向服务链服务节点的功能；

6）SDN控制器与云管理平台对接，可实现业务的自动化部署。

  
**4.4.1 SDN Overlay与Openstack对接**

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiave0m95IIEvQ8EH6SJuXRwkQrchCicEGNzEP1nwVn8dVRryzXrGzh9vSg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
  
                图9 SDN Overlay与Openstack对接

  


如图9所示，与标准的Openstack对接：采用在Neutron Server中安装VCFC插件的方式，接管Openstack网络控制。Openstack定义的插件如表1所示：

                 表1 Openstack定义的插件  
  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavXQg687w2cqibTIUqQlCVaXzfrOzQ0EaMHYS49WjloTzpOoQicrX1Vong/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
  
Openstack插件类似于一个硬件驱动，以网络组件Neutron为例，Neutron本身实现抽象的虚拟网络功能，Neutron先调用插件把虚拟网络下发到VCFC，然后由VCFC下发到具体的设备上。插件可以是核心组件也可以是一项服务：核心插件实现“核心”的Neutron API——二层网络和IP地址管理。服务插件提供“额外”的服务，例如三层路由、负载均衡、VPN、防火墙和计费等。

  


H3C VCFC实现了上述插件，在插件里通过REST API把Nuetron的配置传递给VCFC，VCFC进行网络业务编排通过OpenFlow流表等手段下发到硬件交换机、NFV以及vSwitch上，以实现相应的网络和服务功能。

  


VCFC与H3C CloudOS对接也是采用Neutron插件的方式。  
  


**4.4.2 SDN Overlay与基于Openstack的增强云平台对接  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavR6jhZQw9ic4zv6Zww4JATURpCQIjicptVnsAVU1FMcqicQECMD0tZZh6Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**

          图10 SDNOverlay与基于Openstack的增强云平台对接

  


考虑到Openstack标准版本不一定都能满足用户的需求，很多基于Openstack开发的云平台都在Oenstack基础之上进行了增强开发，以满足自己特定的需求。

  


与这类增强的Openstack版本对接时，基础的网络和安全服务功能仍通过插件形式对接；标准Openstack版本的Nuetron组件未定义的增强功能，如服务链、IPS/AV等等，通过Rest API对接。

**  
4.4.3 SDN Overlay与非Openstack云平台对接  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavba1vQxx4Ud1K53gtONdxUxVyZfQz6JV6OWAHRfMQVzmq02W3Guqdgg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**

                   图11 SDNOverlay与非Openstack云平台对接

  


以CloudStack为例，VCFC与非Openstack云平台的对接通过Rest API进行，H3C提供了完整的用于实现虚拟网络及安全功能的Rest API接口。云平台调用这些接口来实现VM创建、删除、上线等一系列流程。  
  


  


**4.5 服务链在Overlay网络安全中的应用**

**4.5.1 什么是服务链**

数据报文在网络中传递时，首先按特定策略进行流分类，再按照一定顺序经过一组抽象业务功能的节点，完成对应业务功能处理，这种打破了常规网络转发逻辑的方式，称为服务链。

  


服务链常见的服务节点（Service Node）：防火墙（FW）、负载均衡（LB）、入侵检测（IPS）、VPN等。

  


H3C VCF控制器支持集中控制整个服务链的构建与部署，将NFV形态或硬件形态的服务资源抽象为统一的服务资源池，实现服务链的自定义和统一编排。

  


服务链在实现Overlay网络安全方面有独到的优势，服务链方案/VxLAN终结方案除了能够满足Openstack FWaaS、LBaaS定义外，还能提供更灵活的FW/LB编排方案。

**  
4.5.2 Overlay网络服务链节点描述  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavgv9OEfVoFP7icHmsRHCUiaelVhvjNCaLQ6nMTcsmicKFkicicTQGFjbpx6g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**

                  图12 Overlay网络服务链节点描述

  


如图12所示，Overlay网络中的服务链主要由如下几个部件组成：

1）控制器（Controller）：VTEP和ServiceNode上的转发策略都由控制器下发。  


2）服务链接入节点（VTEP1）：通过流分类，确定报文是否需要进入服务链。需要进入服务链，则将报文做VXLAN+服务链封装，转到服务链首节点处理。

3）服务链首节点（SN1）：服务处理后，将用户报文做服务链封装，交给服务链下一个节点。

4）服务链尾节点（SN2）：服务处理后，服务链尾节点需要删除服务链封装，将报文做普通VXLAN封装，转发给目的VTEP。如果SN2不具备根据用户报文寻址能力，需要将用户报文送到网关VTEP3，VTEP3再查询目的VTEP发送。

**  
4.5.3 服务链在Overlay网络安全中的应用  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbu7dpL19adlwTtekHHrEQiavxvkyM8En1hT487S9VKwMZOniaSTotIle83u7StYBTj89WKT5O5I28cA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**

              图13 Overlay网络服务链流程

  


图13是一个基于SDN的服务链流程。SDN Controller实现对于SDN Overlay、NFV设备、vSwitch的统一控制；NFV提供虚拟安全服务节点；vSwitch支持状态防火墙的嵌入式安全；同时SDN Controller提供服务链的自定义和统一编排。我们看一下，假设用户自定义从VM1的VM3的业务流量，必须通过中间的FW和LB等几个环节。通过SDN的服务链功能，业务流量一开始就严格按照控制器的编排顺序经过这组抽象业务功能节点，完成对应业务功能的处理，最终才回到VM3，这就是一个典型的基于SDN的服务链应用方案。



