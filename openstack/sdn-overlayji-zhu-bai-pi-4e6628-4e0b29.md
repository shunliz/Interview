**5 SDN Overlay组网方案设计**

Overlay控制平面架构可以有多种实现方案，例如网络设备之间通过协议分布式交互的方式。而基于VCF控制器的集中式控制的SDN Overlay实现方案，以其易于与计算功能整合的优势，能够更好地使网络与业务目标保持一致，实现Overlay业务全流程的动态部署，在业界逐步成为主流的Overlay部署方案。

  


**5.1 SDN Overlay组网模型  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvjnic8O3UU3pzbawAicRJC6o8ibkfatQAKLu9z7UjicUqOSN7QJoEmkUqibQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**图14 SDNOverlay组网模型

  


如上图所示，H3C的SDN Overlay组网同时支持网络Overlay、主机Overlay和混合Overlay三种组网模型：

  


1）网络Overlay：在这种模型下，所有Overlay设备都是物理设备，服务器无需支持Overlay，这种模型能够支持虚拟化服务器和物理服务器接入；

2）主机Overlay：所有Overlay设备都是虚拟设备，适用服务器全虚拟化的场景，物理网络无需改动；

3）混合Overlay：物理设备和虚拟设备都可以作为Overlay边缘设备，灵活组网，可接入各种形态服务器，可以充分发挥硬件网关的高性能和虚拟网关的业务灵活性。  
  


三种Overlay商用模型都通过VCF控制器集中控制，实现业务流程的下发和处理，应该说这三种Overlay模型都有各自的应用场景。用户可根据自己的需求从上述三种Overlay模型和VLAN VPC方案中选择最适合自己的模型。

**  
5.1.1 网络Overlay**

1. 定位

网络Overlay组网里的服务器可以是多形态，也无需支持Overlay功能，所以网络Overlay的定位主要是网络高性能、与Hypervisor平台无关的Overlay方案。

  


2. 面向客户

网络Overlay主要面向对性能敏感而又对虚拟化平台无特别倾向的客户群。该类客户群的网络管理团队和服务器管理团队的界限一般比较明显。

  


**5.1.2 主机Overlay**

1. 定位

主机Overlay不能接入非虚拟化服务器，所以主机Overlay主要定位是配合VMware、KVM等主流Hypervisor平台的Overlay方案。

  


2. 面向客户

主机Overlay主要面向已经选择了虚拟化平台并且希望对物理网络资源进行利旧的客户。

  


**5.1.3 混合Overlay**

1. 定位

混合Overlay组网灵活，既可以支持虚拟化的服务器，也可以支持利旧的未虚拟化物理服务器，以及必须使用物理服务器提升性能的数据库等业务，所以混合Overlay的主要定位是Overlay整体解决方案，它可以为客户提供自主化、多样化的选择。

  


2. 面向客户

混合Overlay主要面向愿意既要保持虚拟化的灵活性，又需要兼顾对于高性能业务的需求，或者充分利旧服务器的要求，满足客户从传统数据中心向基于SDN的数据中心平滑演进的需求。

**  
**

**5.2 H3C SDN Overlay典型组网**

**5.2.1 网络Overlay**

网络Overlay的隧道封装在物理交换机完成。这种Overlay的优势在于物理网络设备的转发性能比较高，可以支持非虚拟化的物理服务器之间的组网互通。

  


H3C提供的网络Overlay组网方式，支持以下转发模式：

  


1）控制器流转发模式：控制器负责Overlay网络部署、主机信息维护和转发表项下发，即VXLAN L2 GW上的MAC表项由主机上线时控制器下发，VXLAN IP GW上的ARP表项也由控制器在主机上线是自动下发，并由控制器负责代答和广播ARP信息。这种模式下，如果设备和控制器失联，设备会临时切换到自转发状态进行逃生。  
  


2）数据平面自转发模式：控制器负责Overlay网络的灵活部署，转发表项由Overlay网络交换机自学习，即VXLANL2 GW上自学习主机MAC和网关MAC信息，VXLAN IP GW上可以自学习主机ARP信息并在网关组成员内同步。  
  


3）混合转发模式：控制器也可以基于主机上线向VXLANIP GW上下发虚机流表，如果VXLAN IP GW上自学习ARP和控制器下发的虚机流表信息不一样，则以VXLAN IP GW上自学习ARP表项为主，交换机此时触发一次ARP请求，保证控制器和交换机自学习主机信息的正确性和一致性；数据平面自转发模式下ARP广播请求报文在VXLAN网络内广播的同时也会上送控制器，控制器可以做代答，这种模式是华三的一种创新，实现了Overlay网络转发的双保险模型。

  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlv7Cq6S3fC51r25k2MIvXRgxMquZpHY4dBwHQLAJib3Kqv8M3a7LCyOkw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


                         图15网络Overlay

  


在图15的组网中，VCFC集群实现对整个VXLAN网络的总体控制，以及对VNF的生命周期管理和服务链编排；VCFC可以同Openstack、VMware Vcenter、H3Cloud OS等其他第三方云平台，通过插件方式或RESTAPI方式进行对接。

  


物理交换机125X/S98充当VXLAN IP GW，提供Overlay网关功能，实现VXLAN网络和经典网络之间的互通，支持Overlay报文的封装与解封装，并根据内层报文的IP头部进行三层转发，支持跨Overlay网络之间的转发，支持Overlay网络和传统VLAN之间的互通以及Overlay网络与外部网络的互通；H3C S6800充当VTEP，支持Overlay报文的封装与解封装，实现虚拟机接入到VXLAN网络中。

  


Service安全设备属于可选项，包括vFW、vLB、M9000、L5000等设备。东西向支持基于vFW、vLB的服务链；南北向可以由125X串联M9000实现NAT、FW等服务，125X旁挂L5000提供LB服务，由VCFC实现引流。

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvyvx01vElloBnudnex1rs1bjQPISS6QqbmNicZCvsfNPicYBfLzicoF5zA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


                      图16 无状态IP网关

  


如图16所示，在网络Overlay的组网模型中，125X/S98作为Overlay网关功能，考虑到网关的扩容功能，可以采用无状态IP网关方案：

1）VXLAN IP GW实现VXLAN网络与传统网络的互联互通。

2）网关组内的VXLAN IP GW设置相同的VTEP IP地址，设置相同的VNI接口IP地址及MAC地址，VTEP IP地址通过三层路由协议发布到内部网络中。

3）支持多台VXLAN IP GW组成网关组。  
  


无状态网关的业务流向如下：

1）北向业务：VTEP设备通过ECMP（HASH时变换UDP端口号）将VXLAN报文负载均衡到网关组内的不同网关上处理。  


2）南向业务：每个网关都保存所有主机的ARP，并在外部网络上将流量分流给各网关。

3）路由延迟发布确保网关重启和动态加入时不丢包。

  


网络Overlay组网方案有以下优点：

1）更高的网卡和VXLAN性能。  


2）通过TOR实现QoS、ACL，可以实现线速转发。

3）不依赖虚拟化平台，客户可以有更高的组网自由度。

4）可以根据需要自由选择部署分布式或者集中式控制方案。

5）控制面实现可以由H3C高可靠的SDNController集群实现，提高了可靠性和可扩展性，避免了大规模的复杂部署。

6）网关组部署可以实现流量的负载分担和高可靠性传输。

  


**5.2.2 主机Overlay**

主机Overlay将虚拟设备作为Overlay网络的边缘设备和网关设备，Overlay功能纯粹由服务器来实现。主机Overlay方案适用于服务器虚拟化的场景，支持VMware、KVM、CAS等主流Hypervisor平台。主机Overlay的网关和服务节点都可以由服务器承担，成本较低。

  


H3C vSwitch（即S1020V，又称为OVS）以标准的进程和内核态模块方式直接运行在Hypervisor主机上，这也是各开源或者商用虚拟化平台向合作伙伴开放的标准软件部署方式，性能和兼容性可以达到最佳。

  


S1020V上除了实现转发功能，还集成了状态防火墙功能，防火墙功能可以支持四层协议，如TCP/UDP/IP/ICMP等协议。可以基于源IP、目的IP、协议类型（如TCP）、源端口、目的端口的五元组下发规则，可以灵活决定报文是允许还是丢弃。

状态防火墙（DFW）和安全组的区别是，状态防火墙是有方向的，比如VM1和VM2之间互访，状态防火墙可以实现VM1能访问VM2，VM2不能访问VM1这样的需求。

  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvIlrrMD1kxIADYMq4OtbrniccvlBoYZr0MvXCN0B4QeIcdHXJE1YV55Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
图17 vSwitch集成状态防火墙  
  


如图17所示，vSwitch功能按下述方式实现：

1）VCFC通过OVSDB通道将DFW策略下发给S1020V。  


2）S1020V集成DFW功能，依据下发的防火墙策略对端口报文做相应处理。

3）配置DFW策略后，OVS的原有转发流程会以黑盒的形式嵌入到Netfilter框架的报文处理过程中，接收到报文后依据配置的DFW策略在Netfilter的对应阶段调用相应的钩子函数实现对应的防火墙功能。

4）在虚机迁移或删除时，VCFC控制下发相关防火墙策略随即迁移，实现整个数据中心的分布式防火墙功能。  
  


在主机Overlay情况下，H3C vSwitch既承担了VTEP（即VXLAN L2 GW）功能，也可以承担东西向流量三层网关的功能。三层网关同时亦可以由NFV、物理交换机分别承担。vSwitch功能也可以实现Overlay网络内虚机到虚机的跨网段转发。按照VXLAN三层转发实现角色的不同，可以分为以下几个方案：  
  


\(1\)东西向分布式网关转发方案  


如图18所示，在分布式网关情况下，采用多个vSwitch逻辑成一个分布式三层网关，东西向流量无需经过核心设备Overlay层面的转发即可实现东西向流量的跨VXLAN转发，以实现跨网段最短路径转发；南北向的流量仍然会以核心Spine设备作为网关，虚机访问外网时，vSwitch先把报文通过VXLAN网络转发到Spine设备上，Spine设备进行VXLAN解封装后再根据目的IP转发给外部网络。

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvl8RSvXAZJymic3IKdjIVgMr35mMhdibcH9NKy7615WaheRUGawwYy1Kw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
                    图18 东西向分布式网关方案  
  


\(2\)NFV设备VSR做网关方案

VSR做网关的情况下，VXLAN IP GW、VXLAN L2 GW、服务节点都由服务器来实现，如图19所示。

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvpWYRbkVGrnSq6SYO3aD4rvh6ib39ftuia99RX7lmBiahTh8KicCrMgeDeQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
                     图19VSR做网关的主机Overlay方案  
  


VCFC集群实现对整个VXLAN网络的总体控制，以及对VNF的生命周期管理和服务链编排；VCFC可以同Openstack、VMware Vcenter、H3Cloud OS等其他第三方云平台，通过插件方式或REST API方式进行对接。

  


NFV设备VSR充当VXLAN IP GW，提供Overlay网关功能，实现VXLAN网络和经典网络之间的互通，支持Overlay报文的封装与解封装，并根据内层报文的IP头部进行三层转发，支持跨Overlay网络之间的转发，支持Overlay网络和传统VLAN之间的互通以及Overlay网络与外部网络的互通；H3C S1020V充当L2 VTEP，支持Overlay报文的封装与解封装，实现虚拟机接入到VXLAN网络中，其中H3C S1020V支持运行在ESXi、KVM、H3C CAS等多种虚拟化平台上。

  


Service安全设备属于可选项，包括VSR、vFW、vLB等设备，实现东西向和南北向服务链服务节点的功能。

  


\(3\)物理交换机做网关方案

如图20所示，同纯软主机Overlay方案相比，软硬结合主机Overlay方案使用Spine设备做VXLAN IP GW。Spine设备可以使用125-X/98，也可以使用S10500，在使用S10500和S1020V组合的情况下可以实现更低的使用成本。Service安全设备属于可选项，包括vFW、vLB、M9000、L5000等设备。东西向支持基于vFW、vLB的服务链；南北向可以由125-X串联M9000实现NAT、FW等服务，125-X旁挂L5000提供LB服务，由VCFC通过服务链实现引流。

  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvL6D3eDeNOCOicpggOra7QMicAUDBpYCgUyC2CnUSAlR2uiaMMTtvACxWw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
图20物理交换机做网关的主机Overlay方案  
  


主机Overlay组网方案总体来说有以下优点：

1）适用于服务器虚拟化的场景，成本较低。  


2）可以配合客户已有的VMware、Microsoft等主流Hypervisor平台，保护客户已有投资。

3）可以根据需要自由选择部署分布式或者集中式控制方案。

4）控制面实现可以由H3C高可靠的SDNController集群实现，提高了可靠性和可扩展性，避免了大规模的复杂部署。

5）物理交换机做网关的情况下，也同网络Overlay一样可以使用多网关组功能，网关组部署可以实现流量的负载分担和高可靠性传输。

6）vSwitch作为东西向IP网关时，支持分布式网关功能，使虚机迁移后不需要重新配置网关等网络参数，部署简单、灵活。

  
**5.2.3 混合Overlay**

如图21所示，混合Overlay是网络Overlay和主机Overlay的混合组网，可以支持物理服务器和虚拟服务器之间的组网互通。它融合了两种Overlay方案的优点，既可以充分利用虚拟化的低成本优势，又可以发挥硬件GW的转发性能、将非虚拟化设备融入Overlay网络，它可以为客户提供自主化、多样化的选择。

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlv7OenM1a75QFZ7Ne982m9I29v0pFg3V7VRj9sbBHVkNRslyynUbqIug/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
                       图21混合Overlay  
  


VCFC集群实现对整个VXLAN网络的总体控制，以及对VNF的生命周期管理和服务链编排；VCFC可以同Openstack、VMware Vcenter、H3Cloud OS等其他第三方云平台，通过插件方式或REST API方式进行对接。

  


125X/S98充当VXLAN IP GW，提供Overlay网关功能，实现VXLAN网络和经典网络之间的互通，支持Overlay报文的封装与解封装，并根据内层报文的IP头部进行三层转发，支持跨Overlay网络之间的转发，支持Overlay网络和传统VLAN之间的互通以及Overlay网络与外部网络的互通；H3C S6800、H3C S1020V充当VTEP，支持Overlay报文的封装与解封装，实现服务器和虚拟机接入到VXLAN网络中。

  


Service安全设备属于可选项，包括vFW、vLB、M9000、L5000等设备。东西向支持基于vFW、vLB的服务链；南北向可以由125X串联M9000实现NAT、FW等服务，125X旁挂L5000提供LB服务，由VCFC实现引流。

  
**5.2.4 Overlay组网总结  
  
**                      表2 Overlay组网总结  
![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvwnziceq3z8dGQr2BwGKf0ia2bicoT9YQqojibAY319XoD0WJzqH10nBuag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
上述几种Overlay组网均支持和Openstack K版本对接。  
  


**6 SDN Overlay转发流程描述  
  
**

**6.1 SDN Overlay流表建立和发布  
  
**

我们以流转发为例介绍SDN Overlay的转发流程。  
  


**6.1.1 流表建立流程对ARP的处理**

对于虚拟化环境来说，当一个虚拟机需要和另一个虚拟机进行通信时，首先需要通过ARP的广播请求获得对方的MAC地址。由于VXLAN网络复杂，广播流量浪费带宽，所以需要在控制器上实现ARP代答功能。即由控制器对ARP请求报文统一进行应答，而不创建广播流表。

  


ARP代答的大致流程：控制器收到OVS上送的ARP请求报文，做IP-MAC防欺骗处理确认报文合法后，从ARP请求报文中获取目的IP，以目的IP为索引查找全局表获取对应MAC，以查到的MAC作为源MAC构建ARP应答报文，通过Packetout下发给OVS。

  


**6.1.2 Overlay网络到非Overlay网络**

Overlay网络到非Overlay网络的流表建立和路由发布如图22所示：

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvQpGW8X8ZcO5djG06vv5Ouoxia0yY8Vejw8VTY0jW03V5mc41669Drew/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


         图22 Overlay网络到非Overlay网络的流表建立和路由发布

  


创建VM的时候，会同时分配IP和MAC信息。然后VM发送ARP请求报文，该报文会通过Packet-in被上送到控制器。控制器做IP-MAC防欺骗处理确认报文合法后，通过ARP代答功能构建ARP应答报文并通过Packet-out下发。

  


VM收到ARP应答报文后，封装并发送IP首包。OVS收到IP首报后发现没有对应流表，就将该IP首包通过Packet-in上送控制器。控制器通过OpenFlow通道收到Packet-in报文后，判断上送的IP报文的IP-MAC为真实的。然后根据报文中的目的IP查询目的端口，将IP首包直接发送到目的端口，同时生成相应流表下发到OVS。

  


流表下发到OVS后，而后续的IP报文就会根据OVS上的流表进行转发，而不再需要上送控制器。

  
**6.1.3 非Overlay网络到Overlay网络**

非Overlay网络到Overlay网络的流表建立和路由发布如图23所示：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
           图23 非Overlay网络到Overlay网络的流表建立和路由发布

  


创建VM的时候，会同时分配IP、MAC和UUID等信息。VM上线时会触发OVS发送Port Status消息上送控制器，该消息携带VM MAC信息。控制器根据VM MAC查找IP等相关信息，然后携带VM的相关信息通知GW虚机上线。

控制器根据VM上线消息中携带的数据，构造物理机向VM转发报文时使用的流表表项，并下发到VM所在VNI对应网关分组中的所有GW。

  


**6.2 Overlay网络转发流程  
  
**

**1. 识别报文所属VXLAN**

VTEP只有识别出接收到的报文所属的VXLAN，才能对该报文进行正确地处理。  
  


**VXLAN隧道上接收报文的识别：**对于从VXLAN隧道上接收到的VXLAN报文，VTEP根据报文中携带的VNI判断该报文所属的VXLAN。  
  
**本地站点内接收到数据帧的识别：**对于从本地站点中接收到的二层数据帧，VTEP通过以太网服务实例（Service Instance）将数据帧映射到对应的VSI，VSI内创建的VXLAN即为该数据帧所属的VXLAN。  
  


**2. MAC地址学习  
  
**

**本地MAC地址学习：**指本地VTEP连接的本地站点内虚拟机MAC地址的学习。本地MAC地址通过接收到数据帧中的源MAC地址动态学习，即VTEP接收到本地虚拟机发送的数据帧后，判断该数据帧所属的VSI，并将数据帧中的源MAC地址（本地虚拟机的MAC地址）添加到该VSI的MAC地址表中，该MAC地址对应的出接口为接收到数据帧的接口。  
  
**远端MAC地址学习：**指远端VTEP连接的远端站点内虚拟机MAC地址的学习。远端MAC学习时，VTEP从VXLAN隧道上接收到远端VTEP发送的VXLAN报文后，根据VXLAN ID判断报文所属的VXLAN，对报文进行解封装，还原二层数据帧，并将数据帧中的源MAC地址（远端虚拟机的MAC地址）添加到所属VXLAN对应VSI的MAC地址表中，该MAC地址对应的出接口为VXLAN隧道接口。  


  


**6.2.1 Overlay网络到非Overlay网络**

Overlay网络到非Overlay网络的转发流程如图24所示：

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvib59kw8mOaQRiaibgsXJl1O4nHQSOIQ5yYHPlaH2yGxB1RjdSBjnu5ffg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


           图24 Overlay网络到非Overlay网络的转发流程

  


虚拟机构造发送到物理机的报文，目的MAC为OVS的MAC，目的IP为要访问的物理机的IP，报文从虚拟机的虚拟接口发出。

  


OVS接收到虚拟机发送的报文，根据报文中的目的IP匹配OVS上的流表表项。匹配到流表表项后，修改报文的目的MAC为VXLAN-GW的MAC，源MAC为OVS的MAC，并从指定的隧道接口发送。从指定的隧道接口发送报文时，会在报文中添加VXLAN头信息，并封装隧道外层报文头信息。

  


VXLAN-GW从隧道口接收到VXLAN隧道封装报文，隧道自动终结，得到内层报文。然后根据内层报文的目的IP按照FIB（非流表）进行报文三层转发。

  


报文按照传统网络的转发方式继续转发。物理机接收到VXLAN-GW转发的报文，实现虚拟机到物理机的访问。

  


**6.2.2 非Overlay网络到Overlay网络**

非Overlay网络到Overlay网络的转发流程如图25所示：

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvJoQk1AlMO0hLQeql5VoFKxibNB4NbD5f7myx7cKBVg3EdB234Wntia3g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


         图25 非Overlay网络到Overlay网络的转发流程

  


物理机构造发送到虚拟机的报文，在传统网络中通过传统转发方式将报文转发到VXLAN-GW。VXLAN-GW接收该报文时，报文的目的MAC为VXLAN-GW的MAC，目的IP为虚拟机的IP地址，从物理机发送出去的报文为普通报文。

  


VXLAN-GW接收报文，根据报文的入接口VPN，目的IP和目的MAC匹配转发流表。然后从指定的VXLAN隧道口发送。从隧道口发送报文时，根据流表中的信息添加VXLAN头信息，并对报文进行隧道封装。从GW发送报文为封装后的Overlay报文。

  


OVS接收到报文后，隧道自动终结。根据报文VNI和目的IP匹配转发流表。匹配到流表后，从指定的端口发送。从OVS发送的报文为普通报文。

  


根据报文的目的MAC，虚拟机接收到物理机发送的报文，实现物理机到虚拟机的访问。

  


**6.3 Overlay网络虚机迁移**

在虚拟化环境中，虚拟机故障、动态资源调度功能、服务器主机故障或计划内停机等都会造成虚拟机迁移动作的发生。虚拟机的迁移，需要保证迁移虚拟机和其他虚拟机直接的业务不能中断，而且虚拟机对应的网络策略也必须同步迁移。

  


虚拟机迁移及网络策略跟随如图26所示：

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlv3Vq3NsBtfhNbia22jSz30EIHsLjZ2vbicrmy8YlV2OiaHp15qQw8bUkqQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


                 图26 虚拟机迁移及网络策略跟随

  


网络管理员通过虚拟机管理平台下发虚拟机迁移指令，虚拟机管理平台通知控制器预迁移，控制器标记迁移端口，并向源主机和目的主机对应的主备控制器分布发送同步消息，通知迁移的vPort，增加迁移标记。同步完成后，控制器通知虚拟机管理平台可以进行迁移了。

  


虚拟机管理平台收到控制器的通知后，开始迁移，创建VM分配IP等资源并启动VM。启动后目的主机上报端口添加事件，通知给控制器，控制器判断迁移标记，迁移端口，保存新上报端口和旧端口信息。然后控制器向目的主机下发网络策略。

  


源VM和目的VM执行内存拷贝，内存拷贝结束后，源VM关机，目的VM上线。源VM关机后，迁移源主机上报端口删除事件，通知给控制器，控制器判断迁移标记，控制器根据信息删除旧端口信息并同时删除迁移前旧端口对应的流表信息。

  


主控制器完成上述操作后在控制器集群内进行删除端口消息的通知。其他控制器收到删除端口信息后，也删除本控制器的端口信息，同时删除对应端的流表信息。源控制器需要把迁移后新端口通知控制器集群的其他控制器。其他控制器收到迁移后的端口信息，更新端口信息。当控制器重新收到Packet-in报文后，重新触发新的流表生成。

  


**6.4 SDN Overlay升级部署方案**

**6.4.1 SDN Overlay独立分区部署方案  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvibico1byS4P0Y9D6LTaAbj8Fbju736W6MqOyOsZVs8BdNediaia0uC447A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**

          图27 DC增量部署，SDN Overlay独立分区

  


基于对原有数据中心改动尽量少的思路下，可以把SDN Overlay部署在一个独立分区中，作为VXLAN IP GW的核心交换机作为Underlay出口连接到原有网络中，对原有网络无需改动，南北向的安全设备和原有DC共享。

  


场景：在现有数据中心的独立区域部署，通过原有网络互联。

**  
6.4.2 IPGW旁挂部署方案  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvIgv8cFVcVbGQ6tyibsxJYYicPVJADGPV7ibh79BHMejYewBZKSqkib1PUw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**

               图28 DC增量部署，IP GW旁挂

  


考虑到尽量利用原有数据中心空间部署VXLAN网络的情况下，可以采用物理交接机（S125-X/S98）作为VXLAN IP GW旁挂的方案，与经典网络共用核心；而VXLAN网络作为增量部署，对原有网络改动小。

  


场景：利用现有数据中心剩余空间增量部署。

  
**6.4.3 核心升级，SDN Overlay独立分区  
**![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlveSOMcHic2YpLFPaJiagvKbnqlYPoxLDNNwxndZ9AwJPo3mHlbaMAU9qw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**

          图29 核心利旧升级，SDN Overlay独立分区

  


核心设备升级为支持VXLAN IP GW的S125-X，同时作为传统和Overlay网络的核心，原有网络除核心设备外保持不变，充分利旧，保护用户原有投资。安全设备物理上旁挂在核心S125-X上，通过VCFC把VPC流量引流到安全设备进行安全防护。

场景：全新建设数据中心区域，或者升级现有中心的网络核心，原有服务器和网络设备重复利用。

  


6.4.4 Overlay网关弹性扩展升级部署

受制于芯片的限制，单个网关设备支持的租户数量有限，控制器能够动态的将不同租户的隧道建立在不同的Overlay网关上，支持Overlay网关的无状态分布，实现租户流量的负载分担。

  


如图30所示，Overlay网络可以支持Overlay网关随着租户数量增加的扩充，当前最大可以支持超过64K个租户数量，从而提供一个具有弹性扩展能力的Overlay网络架构。

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlv3FDDAT6RYCcCmKEnr59mseiaW2D9kGSek44sBCZuy7cl9VWLrSPYdgQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**图30 Overlay网络弹性扩展

  


6.4.5 多数据中心同一控制器集群部署

![](https://mmbiz.qpic.cn/mmbiz/u6UOjABnicbs505yDg6GBIW9ePcQzicwlvGK9eXRPq6lmuAHuGYOBDibQa1picX8J9n4yKQFQKYHoViac5HiaB8Ozffw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**  
**

             图31 多数据中心同一控制器集群部署

  


控制器跨数据中心部署在多个数据中心，把多个数据中心逻辑上连接为一个数据中心：

1）任一个GW上有全网所有的虚拟机信息，任意一个GW上都可以正确通过Overlay隧道转发到正确的虚拟机。  


2）一个网关组发布相同的VTEP IP地址，每一个数据中心会自动根据最短路径算法，将选择本数据中心的核心设备作为网关，实现本地优先转发。

3）4台控制器的部署，推荐使用2Leader+2Member的主备模式，每个中心各一台Leader和一台Member，4台以上推荐采用多数派/少数派模式。

**  
**

**7 SDN Overlay方案优势总结**

**网络架构方面具有下述明显优势：**

1、应用与位置解耦，网络规模无限弹性扩展；

2、网络虚拟化，实现大规模多租户和业务隔；

3、支持多种Overlay模型，满足场景化需求；

4、跨多中心的网络资源统一池化，随需分配。  
  


**网络安全方面具有下述特点：**

1、各种软硬件安全设备灵活组合，形成统一安全资源池；

2、丰富的安全组合功能，可以充分满足云计算安全合规要求；

3、针对主机，南北和东西向流量，可以实现精细化多层次安全防护；

4、通过服务链，可以实现安全业务的灵活自定义和编排。  
  


**网络业务发放具有下述优点：**

1、支持VPC多租户虚拟网络：基于OpenStack模型，租户相互隔离、互不干扰，各租户可提供独立FW/LB/NAT等服务；

2、网络灵活自定义：租户虚拟网络根据自身需求可灵活自定义，实现对于SDN和NFV的融合控制；

3、网络自动化：业务流程全自动发放，配置自动化下发，业务部署从数天缩短到分钟级；

4、与云无缝对接融合：实现网络、计算与存储的无缝打通，实现云计算业务的自助服务。  
  


**在网络运维上能充分满足客户需求：**

1、支持智能化诊断：全面覆盖的故障自动探测、雷达仿真、故障定位和自动修复；

2、支持流量可视化：应用、虚拟、网络拓扑的统一呈现、资源映射、流量统计、路径和状态感知；

3、支持自动化运维：用户能够自定义网络运维管理能力，实现DC内自定义流量调度、动态流量自动监控分析。

