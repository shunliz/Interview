1

**多云网络，是未来标配**

  


随着云计算的发展，越来越多的企业应用会上云，对于企业中不同的云业务开发者和云业务消费者。如何构建混合云，实现多云网络，构建统一的连通性，统一安全策略，统一管控平台，这是一个非常火热的话题。

  


鉴于运营商网络Telco Cloud极度分布的微小数据中心（Mini/Micro DC），而且主要应用是处理客户流量（vLB/vDPI/vPEC/） 而不是类似OTT提供内容（Content），运营商Telco Cloud更需要深入考虑如何实现多云/混合云的策略。

  


本文介绍如何管理私有云数据中心，构建数据中心互联和混合云解决方案。对于OTT网络架构的深入理解，基本上来源于SIGCOM的白皮书和一些公开视频。

  


![](https://mmbiz.qpic.cn/mmbiz_png/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVraNTiaPxsIKiaDS5eAaY9vlVkMMeeEk0rXzZetmpHA1QpIicG2eN5PQmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


首先很多企业业务都要上云，但是很多敏感数据企业可能需要保留在私有云（企业数据中心）里，或者是租用公有云服务器（IaaS）资源。

  


随着机器学习和AI的发展，越来越多的企业开发者需要利用公有云资源提升算法/数据库创新能力（PaaS）。对于普通的企业用户，会越来越多的采用云化的服务比如微软Office，WorkDay，SAP等企业应用（SaaS）。

  


可以看出，企业IT人员面临要整合私有云，数据中心Fabric，数据中心互联，混合云连接（阿里/AWS等）。急需要统一工具来提供连通性，保证安全策略部署在网络的任意部分，统一的管理控制平台。

  


**▌**为什么需要混合云？

我们以谷歌云为例，最早云计算提供最简单的虚拟机和存储服务。客户需要维护自己的数据库和创建自己的算法代码。最新的云计算公司提供更完整服务，开放最新的数据库技术，提供人工智能机器学习算法\(AI/ML\)，并且提供的完整的数据处理架构。  


  


举一个简单的例子，最近参加谷歌新加坡Google云研讨会，他们介绍如果从头开始一个类似滴滴的Beta项目， 对某个城市例如北京，实时检测司机在地图上的具体地点，乘客的路径规划要求需要匹配最优的司机。而且需要很多前台/后台工作，包括架设很多服务器处理海量请求等等。

  


大家可以想像，如果从头开发，需要多少人来做一个类似的APP？正常情况下大概需要30-50工程师，至少需要6-12个月开发。

  


在谷歌云上已经提供包括地图映射，路径规划算法和信息流处理，并且很容易scale out处理海量请求。利用这种成熟的Cloud开发环境，一个有经验的工程师，二十分钟左右可以做一个类似滴滴雏形的后台系统。

  


可以看到越来越多的中小企业不光采用云服务来获得便宜的计算和存储资源，越来越多的企业采用混合云来利用云公司的先进技术，数据库，人工智能算法，大规模消息处理等等来加速创新。

  


云服务成为创新的催化剂，并且极大提高和简化技术易用性，使得中小企业不需要高端算法工程师，也能很快地推出很酷的主意，解决客户痛点。

  


![](https://mmbiz.qpic.cn/mmbiz_png/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVbM81Mn5s8nzRL3gP0JgtT7Eia46HuoibosMicmCFhicCiceIm6A0mCO0ibGg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


  


  


2

**云网络的架构**

##  

如何构建云网络？云网络由两部分构成。物理underlay网络，传统的路由器，交换机和安全设备提供底层联通。虚拟化的Overlay VPC网络，是在云（公有/私有）上为用户建立一块逻辑隔离的虚拟网络空间。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVwO72hrOvRJY6HmVsLW0jmxJ57DkzeNnocucKLUBmo6Sib7C65yhp4Gw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###  

### **▌什么是VPC**

VPC不是传统的MPLS VPN网络，更类似Linux的Name Space。  


  


在VPC内，有多个可用域（Aviable Zone）。用户可以自由定义网段划分、IP地址和路由策略，可提供网络ACL及安全组的访问控制。一般提供一个Internet GW，做NAT/LB和VXLAN routing。还会提供一个VPC GW，提供IPsec接入VPC互联和企业远程上云（Cloud Onboarding）业务。

  


下图以AWS为例，其他云公司提供类似业务（实现细节有所不同）。

  


云计算需要在大规模服务器物理环境上支持成千上万的客户。最早很多公司采用VLAN进行客户隔离。但是VLAN ID字段只有12 bit，限制网络规模最多支持4K 租户Tenants，这对于海量租户而言肯定是不够的。就不得不把多个客户放到同一个VLAN下面，无法实现真正的客户信息/流量的隔离。

  


现代的VPC一般都是基于VXLAN或类似技术实现的， VXLAN的VNI被扩展成24bit，能够支持1600万个VPC区域，足够支持云计算海量客户增长。

  


VXLAN是一种通用的Overlay封装技术。将原始MAC/L3报文封装成UDP包，可以很方便的跨越三层网络传输二、三层内容。能够让用户构建的分布在不同物理服务器Server/不同数据中心的VPC里面的客户IP/MAC数据被封装进Server可寻址的IP地址，进而提供Scale Out云计算解决方案。

  


软件定义Overlay网络很容易快速迭代新的算法，给云计算带来了越来越多的网络创新。  


  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVZ6ERcsc8pAmtUEkHiaZvCDPXBqm3TzedZoocR2Te5hEONSnB9AuibmmA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


### **▌VPC SDN控制器**

对于云计算来说，网络中有成百上万级别的主机VM，Containers，Serverless服务。每个终端都需要相应的IP地址，策略组，负载均衡，Anti-DDoS, VPN隔离等。  


  


以Google 为例，为维护全球网络，SDN控制器需要在左右很短时间内即180ms左右，在全球DC网络部署10万级别VM。

  


![](https://mmbiz.qpic.cn/mmbiz_png/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVic4hHfBxs88GoPD97kfLMhrb1KLVzXUqqMcD615uNUd2xm6ZXpOzcPw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


Google采用Divide and Conquer方式在全球部署多个区域region的Fabric Manager来管理Jupiter数据中心TOR/Spine交换机。同时每个region采用多个仙女座（Andromeda）控制器来进行网络虚拟化，管理虚拟机VM和容器。

  


通过Openflow Front Endpoint（OFE）来下发转发流表。VM之间缺省流量会经过Hoverboard（跳板），同时对于大象流（Elephant Flow），Andromeda提供bypass offload卸载创建VM-VM的直连tunnel。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVhibhYfcfLBMmJUib9ficnbcKXj3t63WDsPIcYtibs7bBibic3cmtGBf13Zibg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


在数据中心的每一个Server上创建一个虚拟的Andromeda转发面，通过IPinIP来构造Overlay网络，虚拟多个不同的VPN，把VM/Containers映射到不同的虚拟网络。 每个转发Forwarder可以提供Load balance/DDos/ACL/VPN能力。

  


![](https://mmbiz.qpic.cn/mmbiz_png/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVsKUPicouHicdEUFic9yLAOzvOF7ibCAXS3QiangibwDgWcialbEoFfHPQKAGw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


业界其他云公司，例如AWS/Azure都提供类似功能。AWS物理机之间通过IPinIP封装在VPC头里面，提供L2/L3跨越数据中心/Region的服务器虚拟化。并且采用Blackfoot作为VPC和外界通讯的VPC网关。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVl1HnIzNnNCmbKGUl5e9GRKyDibGxMDrK3WjrmK8v5Q2jCE17s6DRFgA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


开源Contrail SDN控制器实现跟Google 仙女座（Andromeda） 同样的功能。

  


Contrail基本设计思想是：每个数据中心服务器虚拟化出来一个vRouter, 通过vRouter来实现类似EVPN/L3VPN功能。多个VM/Container会连接到这个vRouter的不同的Tenent VRF。同时vRouter之间采用GRE/UDP/VXLAN做外层隧道, 采用内层标签来标识不同的VRF。 vRouter相当于传统L3VPN和EVPN的一个vPE功能。vPE用户侧不再接入CE设备，而是为VM/Container提供连接性。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVuzhwc8tZKsoAVQH0COy1VUV9HSLia2txErMkhDH1Sq8BlfIwoMjsafQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


通过Orchestrator在Server上部署一个VM或者容器：

> 1、首先给VM/Container POD分配IP地址，DNS等等信息。在vRouter上创建VRF/EVI，RT/RD等信息，上送回传到Contrail控制器。vRouter之间不需要协议通讯，vRouter仅仅跟Contrail控制器进行控制平面的通信。
>
> 2、生成L3VPN/EVPN转发表， 控制器知道现存的多个vRouter可能要跟新创建的vRouter共享相同的VRF/EVI，并且需要互相通讯，就通过XMPP来下发转发表信息（BGP NLRI内嵌到XMPP消息里）到另一台服务器上的vRouter。vRouter之间仅仅建立转发平面的动态隧道。这样Server之间的VM/Containers就可以通讯了。
>
> 3、控制器通过BGP/Netconf来通知GW Router来自动发布VM/Container的 IP prefix.这样Openstack/vCenter创建的VM/Container就可以被外界来使用了。

  


简单的来讲，如果客户有一万台服务器，部署了Contrail之后：

> 1、Contrail控制器相当于传统的路由器控制平面，承担计算Overlay拓扑，数据通道Tunnel建立等工作。相当于传统VPN的RR。
>
> 2、数据中心的Leaf/Spine交换机提供IP Clos无阻塞交换，相当于一个虚拟的交换矩阵，为控制器和vRouter之间提供背板交换。
>
> 3、vRouter/vSwitch跟Controller建立控制关系，vRouter之间建立数据tunnel。每个vRouter相当于传统路由器的一个板卡。
>
> 4、部署Contrail SDN控制器之后，数据中心服务器里的很多vRouter一起组成了巨大的虚拟路由器系统\(Network as a Router\)。为数以万计的虚拟机/容器提供多租户隔离的VPN网络连通。

  


**▌VPC Fabric控制器**

对于MSDC（大规模数据中心），不仅仅需要SDN控制器去管理vRouter，也需要全套工具管理路由器交换机，构造DC Fabric和DCI（数据中心互联）Fabric。  


  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVibF7DLyJS5Jb1GWhLpb5o4GjHvpnANzdLE4Co29Lbdk1LHZvs1viaGLQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


对于Cloud/DC/DCI fabric，针对不同拓扑（P2P/CLOS/HUB&Spoke）和不同设备（vRouter/Switch/Router）。需要采用不同技术来实现Fabric自动部署和监控。

  


关于DC fabric Day1自动化配置部署，大部分厂家采用非常流行的Ansible来部署EVPN/VXLAN. 如下图所示，最后产生针对不同设备的配置Conf。

  


首先要定义角色（Role），组（group），主机（host），拓扑（topo），等Yaml文件，然后采用Playbook去自动产生配置，并下发到每个Leaf/Spine交换机。Contrail Fabric Management（CFM）对Ansible也提供统一的GUI来进行自动化配置。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVOn7hdKvJibZVBo4JPC06CZPc5RKBsFgia2mthobVGIH6rk5FXBuW6RrQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


解决了配置的基本问题，同时CFM还可以支持。完整的ZTP/ZTR（自动配置），软件升级，拓扑发现，并且自动配置收集Telemetry遥测信息进行网络故障诊断。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVHAzWebCZnyQN9aDmtJveUkPYiczWPLHGn6v01SPLDPy9lu18jjBpfDQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


对于数据中心互联DCI（Data Center Interconnect），CFM也采用类似的配置管理方式。配置对象变为GW router，配置技术以MPLS/VPN和IP/Optical为主。

  


  


3

**多云网络Multi-Cloud**

  


### **▌VPC网关GW**

以AWS VPC网络为例，一般会有一个IGW（Internet GW）提供缺省路由NAT/LB等功能。还会提供一个VGW来提供IPSec互联。比如AWS的BlackFoot提供VGW IPsec功能，还能做Direct Connect（Colo数据中心互联）并且支持AWS 报文头（IPinIP）连接VPC网络。

![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVRf9LbRUiagz5Ia0ZHcbXAN6RkLA6IWJQQMj4fo6AxibL8t1pPuceoKnQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

企业接入VPC网络可以有以下几种方式：

> 1、Direct Connect，通过在IXP/Colo数据中心，通过一对儿云路由器和企业路由器上的VLAN接口，连接云和企业，并且支持BGP路由分发。可以提供高达80Gbps的大带宽接入。一些云公司还提供通过运营商/IXP的MPLS VPN网络接入云。
>
> 2、Private Link，满足多个VPC互访要求，企业可以在VPC上注册一个私有链路（Private Link），在VPC上提供Elastic Network Interfaces（ENI），其他VPC可以通过白名单方式访问企业VPC资源。
>
> 3、IPsec VPN, 一般通过IPsec连接到Cloud的VGW上。云提供客户CPE的配置模版。但并不管理客户的CPE设备。提供大概1Gbps的接入带宽。
>
> 4、SDWAN接入，近期很多云公司提供一个CPE小盒子。提供自动化部署，帮助企业更好的上云（Cloud Onboarding）。

###  

  


### **▌VPC对等互联**

### 企业客户上云，业务需要一定程度的隔离。比如需要为研发，测试，运维团队申请多个VPC，进行有效的灰度发布和快速迭代。也可能先在北京上海VPC部署，随着业务增长，逐渐增加海外节点等VPC。这就需要实现VPC在同一/不同region的互联。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVMGNJm6EFXvsh7LFz3oCNFbJkGWDWODtvCnPMhfFKicpBQwHIS5cLI3w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


VPC互联，一般有两种方式，IGW互联和VGW互联。下面介绍一下技术方案的优缺点。

  


**■ VPC Peering内部互联**

VPC-A内部采用VXLAN200，VPC-B采用VXLAN300隔离。每个VPC都通过自己的IGW来访问Internet，分别有自己的路由表。

  


对于两个VPC Peering（互联）需要IP地址不要重叠。在VPC-A上创建一个VXLAN VTEP tunnel到对端VPC-B。同时把Prefix B指向VPC-B的VXLAN tunnel。VPC-B收到VXLAN 100的报文，会经过VXLAN Routing路由查表，找到相应的VPC-B里面的Host，封装转换成VXLAN-200发送给相应的VM /容器。

  


具体实现方式可以有Symmetric/Asymmetric两种方式。Symmetric需要两个VPC翻译本地VXLAN到第三个VXLAN VNI，带来管理和配置的复杂问题。一般采用Asymmetric方式，由VPC做本地和远端VXLAN的routing/翻译。  


  


通过VPC peering（IGW互联）客户可以跨越多个区域部署多个不同的VPC。比如可以把Asia Pacific \(Singapore\)和US West \(Northern California\)的VPC，内部打通。

  


**■ VGW Internet互联**

VPC peering互联方式，利用云公司内部网络，在网络传输上不需要加密，性能比较好。一些客户需要通过internet连入云，或者需要在不同VPC之间部署复杂的访问策略。比如Extranet/Transit VPC方式。在VGW上一般可以采用IPSec接入VPC。还可以通过BGP over IPsec提供动态路由分发。

  


**▌SDWAN接入VPC**

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVZpyzH6gyYVCRp7u1xyTia8fGRYlLO0Lgkg0H8MuePMWRnKRE3Ib8gCA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


对于企业网接入Cloud，最早云管理到PoP点。企业需要自带设备在IXP/SP互联节点接入VPC。最近多家云公司纷纷推出SDWAN接入方式，极大的方便了中小企业上云。

> 网上下单，云公司快递发送小盒子去分支机构或者企业总部。  
>
>
> 加电，扫码，上网，自动注册，自动下载配置到小盒子。
>
> 提供统一的云管平台管理VPC和SDWAN CPE设备。
>
> 提供本地流量的分流，访问VPC，访问Office365和视频流量走不同的路径。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVxRia1ia8ylcR0lPicZ5RdcK8rW9Xx9RiaWKBXey4LWfiaZKwqOyo1MRncAA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


2018年7月，微软宣布开始最新的Virtual WAN\(SDWAN\)解决方案试运行。企业用户可以通过SDWAN（CPE）设备接入微软的不同VPC region，访问VPC里的各种资源，极大的加速了分公司/个人基于云产品的DevOps。

  


同时还提供客户基于微软全球骨干网构造精品企业网，提供Internet无法达到的跨越多个运营商SP的SLA，可以基于微软全球骨干网构建精品视频会议网，支持各种高带宽低时延的新型业务。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVHVvGRMc4fxWicCIEia0Zib2Pr8C8TnRRdib6VxIuRVIbY7sYw0T6KsZGdg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


从上面的微软Virtual WAN控制台可见。客户可以自行创建新的SDWAN Site，上线新的SDWAN CPE盒子。管理Site/Hub（微软POP点）之间的VPN连接，提供自动化CPE配置脚本。还能够配置统一接入策略和监控网络运行情况。

  


**▌Multi-Cloud/IaaS**

### 企业现有的数据中心和分支机构要接入云，还要维护相同的策略控制和安全接入，同时要实现虚拟机/容器在私有数据中心和公有云之间的双向流动。需要SDN控制器来管理私有云，同时能自动创建VPC。并且在VPC里面用Contrail vRouter去替换VGW功能，实现全网的连通性。由于VGW由私有数据中心SDN控制器创建，所以可以保持一致的策略和访问控制。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVIoDJIWPjm4dibwTPZxJRIxNOudZciaibCMJaMH8fOnnrYhnDombKSh8sA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


一些Startup创业公司和开源组织提供多云的管理。比如Terraform/Istio就支持动态创建VPC,并且能指定VGW运行不同的软件Image。比如采用如下代码就可以在AWS里面创建一个VPC，并且指定VPC GW连接到哪个DC的CPE设备上。Terraform支持AWS/GCP/Azure和国内的阿里云/腾讯云等。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVnq0ccjb3wg1BrkZl9zupoWUeASduibheINiaTPFFiaTjZOqpbMzOY34Jg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVP0yosZXTax5995Dvveo2qeOWEePMcuOtwweODTic1MCTyUXAuLv1ceg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


在私有云/数据中心采用Contrail SDN控制器来管理VM/Container,同时

> 1、采用Terraform或者Cloud GW API动态创建VPC，并用Contrail vRouter替代VGW功能
>
> 2、在分支机构（SDWAN），私有云/数据中心和公有云之间建立安全低时延的IPsec网络连接。并且自动配置BGPoIPsec来双向发布VPC和private Cloud的路由信息。
>
> 3、Contrail SDN控制器来统一下发控制策略和隔离，并且驱动Cloud API GW/Terraform来改变VPC网络Prefix和安全组策略。Contrail还调用 Cloud API/Terraform来实时查看VPC路由和策略变更。
>
> VM/Container可以在私有云和公有云之间动态迁移。

  


##  

4

**总结**

##  

## 迄今业界最成功的SDN企业就是Google，采用四个不同的SDN控制器来优化网络。

> ## 1、Andromeda虚拟化Overlay控制器。
>
> ## 2、Jupiter数据中心Fabric控制器。
>
> ## 3、B4 DCI数据中心广域网互联控制器。 
>
> ## 4、Espresso基于BGP的广域网Peering控制器 。

##  

## Google有强大的研发能力，可以定制化云主机管理软件，定制化DCI交换机，定制化路由协议。但是Google的云计算网络部分不一定适用于第二家互联网公司，也不一定适用于运营商和企业客户。

  


Juniper公司贡献开源SDN控制器 Tungsten Fabric（Contrail）https://tungsten.io作为Linux Foundation的一个重要SDN开源项目，Contrail SDN 控制器能提供相当于Google的Andromeda和Jupiter fabric 控制器同样的功能。可以更好的帮助企业上云（Cloud Onboarding）自动管理分支机构远程接入云VPC，极大的简化客户私有云和公有云多云的自动化部署，是客户转型云计算的有力的工具。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/3KXMWoMk9CAbySB4IcKHJ13VITqefqDVOcwFezpfekKlAKWwzJr4acpjG8UsFrvSe2SmIlaCsmzVd2RfGNmwgg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


为了管理多云网络，需要从最早的云主机管理, 引入了更多云网融合的新功能。

  


云计算从提供最早的公有云主机，竞争逐渐外延到管理企业私有数据中心/云和企业分支机构。云计算的SDN和高度自动化引入会带来企业网的全新变革。谁能更有效帮助企业上云（Cloud Onboarding）谁就能在云计算中占据先机。

