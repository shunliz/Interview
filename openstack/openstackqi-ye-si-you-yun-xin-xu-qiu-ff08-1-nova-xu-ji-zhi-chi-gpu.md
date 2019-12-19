本系列会介绍OpenStack 企业私有云的几个需求：  


* GPU 支持

* 自动扩展（Auto-scaling）支持

* 混合云（Hybrid cloud）支持

* 物理机（Bare metal）支持

* CDN 支持

* 企业负载均衡器（F5）支持

* 大规模扩展性（100个计算节点）支持

* 商业SDN控制器支持

  

内容比较多，很多东西也没有确定的内容。想到哪就写到哪吧。先从 GPU 支持开始。

  


## 1. 基础知识

### 1.1 VGA（图像显示卡），Graphics Card（图形加速卡），Video Card（视频加速卡），3D Accelerator Card 和 GPU（图形处理器）

  


对这些概念之前也没怎么了解，这次正好自己梳理一下。从一篇古老的文章中，找到所谓的显卡从 VGA 到 GPU 发展史：

  


1. 第一代显卡：支持 256 色显示的 VGA Card，1988年。VGA Card的唯一功能就是输出图像，真正的图形运算全部依赖CPU，所以当微软 Windows 操作系统出现后，PC 的 CPU 就开始不堪重负了。

2. 第二代显卡：Graphics Card，支持 Windows 图形加速，1991年，通过一颗专用的芯片来处理图形运算，从而将 CPU 部分解放了出来，让Windows界面运行起来非常流畅，从此图形化操作系统资源消耗大降、实用性大增。为了与单纯具备显示功能的 VGA Card 相区别，具备图形处理能的显卡被称为Graphics Card，也就是图形加速卡，它加速了Windows的普及，让PC走进了图形化界面时代。

3. 第三代显卡：Video Card，支持视频加速，1994年。为了与单纯具备图形加速能力的 Graphics Card 相区别，具备视频辅助解码的显卡被称为 Video Card，也就是视频加速卡。

4. 第四代显卡：3D Accelerator Card，1994年。后起之秀 NVIDIA 则凭借性能强大的单芯片TNT和TNT2系列显卡超越3DFX 从而脱颖而出。

5. 第五代显卡：GPU 图形处理器，支持硬件T&L，1999年。GeForce 256是一款划时代的产品，NVIDIA 将其称为第一款GPU（Graphic Processing Unit，图形处理器），显示芯片上升到了与CPU（Center Processing Unit，中央处理器）同样的高度。GeForce 256是被作为一个图形处理单元\(GPU\)来设计的，GPU是一个单芯片处理器。它有完整的转换、光照、三角形设置和渲染引擎\(分别为:Transform、Lighting、Setup、Rendering\)等四种3D处理引擎，一些以前必须由CPU来完成的图形运算工作现在可以由GeForce256 GPU芯片独立完成，大多数情况下具有完整的传输和光照相引擎的GPU运算速度比CPU快2-4倍，同时也有效地减轻了CPU的浮点运算负担，减少了对CPU的依赖性。

6. 未来的显卡：GPU 接管更多CPU的功能，支持更多的功能，包括几何着色、物理加速、高清解码、科学计算等。

  

简化一下：

1. VGA Card：640×480分辨率彩色图形显示，单纯的输出图像

2. Graphics Card：支持图形界面加速，减轻CPU负担

3. Video Card：支持视频解码加速，减轻CPU负担

4. 3D Accelerator Card：支持3D图形渲染，3D技术走向普及

5. GPU：支持坐标转换和光源处理，消除3D渲染的瓶颈

###  

### 1.2 GPU 与 CPU



从上面的介绍我们知道，GPU 表示 Graphics Processing Unit，即图像处理单元。一开始的时候GPU 主要用于 3D 游戏的渲染，但是现在GPU已经广泛用于加速计算性负载，比如金融模型计算、科学研究以及石油和天然气开发等。从架构上看，CPU 是由若干核（core）和许多的缓存（cache memory）组成，因此CPU可以并行处理若干线程。相对地，GPU是由几百个核组成，因此可以并发处理数千个线程。尽管 GPU 的内核数目远远超过 CPU，但是它的每个核的处理能力远小于CPU的核，而且不具有现代操作系统的所需要的一些特性，GPU 并不合适用于处理普通的计算。它们更多地用于计算消耗性操作，比如视频处理和物理仿真等。这个文档 中包含了目前支持 GPU 的应用列表。  


  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYye6nlkia7s0ic0PpFclcsibstEiaqc7nC3y4opNrOWViamQ2hTKWiaKITuahQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYyJvchLCsicCECelJFDYtawaJhjMtmgjxbQPPK8rIY1Hqnof6bf82OSWQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

                               （GPU 和 CPU 对比）                                                                     

  


（GPU 应用软件栈）

注：Direct3D（简称：D3D）是微软公司在Microsoft Windows操作系统上所开发的一套3D绘图编程界面，是DirectX的一部分，目前广为各家显卡所支持。与OpenGL同为电脑绘图软件和电脑游戏最常使用的两套绘图编程界面之一。

###  

### 1.3 在虚机内使用 GPU 的几种方式 （GPU 虚拟化）

####  

#### 1.3.1 集中 GPU 虚拟化实现技术

  


（1）API Remoting （远程API）

![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYy7bTuwgs1HGVG4cS9h3u4icWMS3Ol3LS7VUaLnsKLY8qNFJb3cbkhVdw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYy6m1CPblNfLHiaIQUtf4K56ibCgw7HFmNHtMwCZZkpzuRCwB95o6W6TzQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



远程API 方法分为前端和后端两个部分。前端以动态库的形式被虚拟机中的CUDA程序加载，后端则是运行在宿主机中的一个程序。在这种机制下，首先由前端将虚拟机中的CUDA API重写，将API的名称和相应参数传递给后端。然后后端为前端每个CUDA应用程序创建一个进程，在该进程中转 换来自前端重写后的API，获得API的名称和参数，接着使用宿主机上真实的GPU硬件设备执行相应的API，最后将 API执行结果返回给前端。  

  这种方法需要进行大量虚拟机与宿主机之间的数据传输，导致GPU虚拟化的性能严重下降。在CUDA程序规模较小时，这些GPU虚拟化框架的性能下降并不太明显。但在进行实际应用中的高性能计算时性能下降非常明显。

  Parallels Desktop 所使用的技术和该技术非常接近。

  


（2） Device Emulation （GPU 设备仿真）

  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYyJagNySYCb95pQ1V4ZRo2OW3SU8CADdMs3rFC1ugInqicErhNufYXRhw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


在客户机中提供一个仿真虚拟GPU，客户机上应用对它的调用都会被仿真层转化为对Hypervisor上物理 GPU 的调用。

  


（3） 两种 Pass-through （透传）

  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYypcHhicxxE7Lyv74GgP0E0gwiaYUS7v2hNibOiczZiaf3c4toNUKcZrq6JBg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYyEYpzA0hQ8EK04rEELxpKknNZ53NCibu5ILlCw7Wy3iatPsNMuQx72ylQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

（Fixed pass-through：固定透传，一个 GPU 只能给一个虚机使用）     （Mediated pass-through：中介式透传，一个 GPU 可以给多个虚机使用）

  


（4）全虚拟化

  


全虚拟化方案中，每个虚机拥有一个虚拟的GPU实例，多个虚机共享一个物理 GPU。下图是 Intel 的 GPU 全虚拟化示意图：

![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYyYax2QEfjYic4PUxTKURGQ01hHCyTicsNHjz9usnXg91Oobu0HNG96DnA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

####  

#### 1.3.2 Intel 实现的 GPU 虚拟化

  


Intel 有如下几种 GPU 虚拟化技术：

![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYy4wUmQVicgln7MlIvINLgibEiaL1AoBcD0V3PUOEGJZhqTlY1DfZrUyl1Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


三者之间的比较：

![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYyiaGUwVHmJBia7lVhTXBOXibUgZbx17PlcLbwdiby1UhqyxRc82LUmc0XJw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


第三个最先进，它支持 GPU 全虚拟化，以及在多个虚机之间共享一个物理GPU。目前已经在 Xen 中完整地支持该技术（XenGT 项目）。但是在 KVM 中，KVMGT 是在 KVM 内的实现，但是直到 2014/12月 Intel 才出一个 KVMGT 版本，目前仍然处于初级阶段（资料来源）。

  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYy4s0gibMmufmcZUpHfCTtjZZawxY9aEywoSibInA1JCbZ6Kf4RIz2FUfw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

####  

#### 1.3.3 Nvidia 实现的 GPU 全虚拟化

跟 Intel 的情况差不多，Nvidia K1/K2 GPU 也有 GPU 全虚拟化技术，但是目前也是不支持 KVM，而是只支持几家主流的虚拟化软件比如 Hyper-V 和 VMware 等\(资料来源\)。

  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYyvW0EWht0gubVWIa6kmiapGVQ6WKicc3LCNmeiaQFialPeuQeBOqFEnnkgQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 

### 1.4 透传（Pass-through）技术

  从上面的介绍可以看出，目前主要的 GPU厂商包括 Intel 和 Nvidia 的全虚拟化方案主要还是针对某几种商业虚拟化软件比如 Hyper-V 和 VMware 等，对于 KVM 的支持要么没有，要么还处于早期阶段。鉴于 KVM 在 OpenStack 计算虚拟化层的地位，OpenStack Nova 支持的也只是 GPU 透传给客户机。QEMU/KVM GPU 透传主要有两种实现方式。

####  

#### 1.4.1 两种实现方式

（1）pci-assign 方式

  主流地，QEMU/KVM 使用 PCI Assign 技术将KVM主机上的一个 PCI 设备比如 GPU 和 网卡等直接分配给一个虚机。这技术需要 Intel VT-d 或者 AMD IOMMU 硬件支持。下面是一个 Intel 平台上的实现步骤的例子：

![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYySNpxLOngPz0A0lF2uh0h40Zb6l0zvb5X2QV86yPs3M6ibpic0BkWkCbA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

\# Boot kernel with 'intel\_iommu=on'\# Unbind driver from the device and bind 'pci-stub' to it  
echo "168c 0030" &gt; /sys/bus/pci/drivers/pci-stub/new\_id  
echo 0000:0b:00.0 &gt; /sys/bus/pci/devices/0000:0b:00.0/driver/unbind  
echo 0000:0b:00.0 &gt; /sys/bus/pci/drivers/pci-stub/bind  
  
\# Then just run  
sudo qemu-system-i386 -m 1024 \    -device pci-assign,host=0b:00.0,rombar=0 \    -enable-kvm \    -kernel $KERNEL \    -hda $DISK \    -boot c \    -append "root=/dev/sda rw"  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYySNpxLOngPz0A0lF2uh0h40Zb6l0zvb5X2QV86yPs3M6ibpic0BkWkCbA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

  


（2） VFIO 方式

  VFIO 在 Linux kernel3.6/qemu1.4 以后支持，目前只支持 PCI 设备。VFIO 是一套用户态驱动框架，提供两种基本服务：向用户态提供设备访问接口 和 向用户态提供配置IOMMU 接口。 它第一次向用户态开放了 IOMMU 接口，能完全在用户态配置 IOMMU，将 DMA 地址空间映射进而限制在进程虚拟地址空间之内。

  


  VFIO 可以用于实现高效的用户态驱动。在虚拟化场景可以用于 PCI 设备透传。通过用户态配置IOMMU接口，可以将DMA地址空间映射限制在进程虚拟空间中，这对高性能驱动和虚拟化场景device passthrough尤其重要。相对于传统方式，VFIO对UEFI支持更好。VFIO 技术实现了用户空间直接访问设备。无须root特权，更安全，功能更多。



它对环境有如下要求：

* AMD-Vi or Intel VT-d capable hardware

* Linux 3.6+ host

* CONFIG\_VFIO\_IOMMU\_TYPE1=m

* CONFIG\_VFIO=m

* CONFIG\_VFIO\_PCI=m

* modprobe vfio-pci

* Qemu 92e1fb5e+ \(1.3 development tree\)

####  

#### 1.4.2 透传技术的局限性

  透传技术能带来几乎和物理设备同等的性能，但是它也带来了一些局限性。设备透传带来的一个问题体现在实时迁移方面。实时迁移 是指一个 VM 在迁移到一个新的物理主机期间暂停迁移，然后又继续迁移，该 VM 在这个时间点上重新启动。实时迁移是在一个物理主机网络上支持负载平衡的一个很好的特性，但使用透传设备时它会产生问题。PCI 热插拔（有几个关于它的规范）就是需要解决的一个问题。PCI 热插拔允许 PCI 设备从一个给定内核进出，这很理想 — 特别是将 VM 迁移到新主机上的管理程序时（设备需要在这个新管理程序中拔出然后再插入）。当设备被模拟（比如虚拟网络适配器）时，模拟提供一个抽象层以抽象物理硬件。这样，一个虚拟网络适配器可以在该 VM 内轻松迁移（这个 VM 还得到 Linux® 绑定驱动程序的支持，该驱动程序支持将多个逻辑网络适配器绑定到相同的接口上）。

  


更详细的介绍，可以阅读文档 http://www.linux-kvm.org/images/b/b4/2012-forum-VFIO.pdf。

  


### 1.5 公有云上的 GPU 支持

  


（1）阿里云 GPU 物理机，用于高新能计算

阿里云（来源）为高性能计算提供带 GPU 的物理机，配置如下：

* 双路Xeon E5－2650v2 2.6GHz

* 128G 内存

* Raid5 13T 数据盘

* K40M x2 GPU

该物理机的单机峰值计算能力可达每秒11万亿次单精度浮点运算。

  


（2）Amazon GPU 虚拟机，可用于 3D 应用程序流、机器学习、视频编码和其他服务器端图形 或 GPU 计算工作等负载，包括 Linux GPU 虚机和 Windows GPU 虚机。虚拟机配置如下：

![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYydibBxZg5UMkBrsea8QG4acJWMo0YCGbBl5pFjCQJpY46QmIpQFYQM9g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)（资料来源）

##  

## 2. OpenStack Nova（QEMU/KVM） 对 GPU 的支持

  OpenStack 官网的 https://wiki.openstack.org/wiki/Pci\_passthrough 文章说明了 Nova 对 GPU 的支持。本章节就从配置和代码等角度来分析Nova是如何支持 GPU 透传的。

  


### 2.1 环境准备

可以使用 lspci 命令来获取 GPU PCI 设备：

\# lspci -nn \| grep NVI85:00.0 VGA compatible controller \[0300\]: NVIDIA Corporation GK104GL \[GRID K2\] \[10de:11bf\] \(rev a1\)86:00.0 VGA compatible controller \[0300\]: NVIDIA Corporation GK104GL \[GRID K2\] \[10de:11bf\] \(rev a

  


1\) 输出中各个值的说明：

| 输出值 | 含义 | 详细解释 |
| :--- | :--- | :--- |
| "85:00.0" 和 “86::00.0” | 以 ”bus:slot.func“ 格式来唯一标识一个 PCI 功能设备 | 唯一定位一个 PCI 设备的虚拟功能，可以是一个物理设备，也可以是一个多功能设备的功能设备，一个多功能设备可以最多有8个功能。总线号（bus）：  从系统中的256条总线中选择一条，0--255。设备号（slot）：  在一条给定的总线上选择32个设备中的一个。0--31。功能号（func）：  选择多功能设备中的某一个功能，有八种功能，0--7。 PCI规范规定，功能0是必须实现的。 |
| ”0300“ | PCI 设备类型 | 指 PCI 设备的类型，来自不同厂商的同一类设备的类型码可以是相同的。 |
| “10de” | 供应商识别字段（Vendor ID） | 该字段用一标明设备的制造者。一个有效的供应商标识由 PCI SIG 来分配，以保证它的唯一性。Intel 的 ID 为 0x8086，Nvidia 的 ID 为 0x10de |
| “11bf” | 设备识别字段（Device ID） | 用以标明特定的设备，具体代码由供应商来分配。本例中表示的是 GPU 图形卡的设备 ID。 |
| “a1” |  版本识别字段（Revision ID） | 用来指定一个设备特有的版本识别代码，其值由供应商提供 |

下图说明了 PCI 域、总线、设备等概念之间的联系（lspci 的输出没有标明域，但对于一台 PC 而言，一般只有一个域，即0号域。）：

![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYysD6n400sOC4XiaABEaMJibMaf2u1CsciaTEJ4kTKqxTrxa8uMywQ83ORQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)（来源）

###  

### 2.2 KVM主机和Nova 配置

#### 2.2.1 修改 Nova 配置

| 节点 | 配置文件 | 需要修改的配置 | 说明 |
| :--- | :--- | :--- | :--- |
| 控制（API）节点 | /etc/nova/nova.conf | 添加配置项：pci\_alias={"vendor\_id":"10de", "product\_id":"11bf", "name":"a1"} | 这是为了在 Nova flavor 中可以很方便的使用 alias的需要而添加的。 |
| 计算节点 | /etc/nova/nova.conf | 添加配置项：pci\_passthrough\_whitelist=\[{ "vendor\_id":"10de","product\_id":"11bf"}\] | 并不是一个 计算节点上的所有 PCI 设备都运行被分配给客户机，云管理员可以使用该配置项来指定可以分配的 PCI 设备的范围。在 Nova 中，PCI 设备可以分为几类：1、普通的 PCI 设备 2. SR-IOV PF 设备 3. SR-IOV VF 设备。通常 SR-IOV PF 设备都是不可以直接分配给客户机的，因此，该配置项是来指定可以分配的 “普通 PCI 设备”以及 “SR-IOV VF” 的范围。 |
| Nova scheduler 节点 | /etc/nova/nova.conf | scheduler\_available\_filters=nova.scheduler.filters.all\_filtersscheduler\_available\_filters=nova.scheduler.filters.pci\_passthrough\_filter.PciPassthroughFilter修改配置项：scheduler\_default\_filters=RamFilter,ComputeFilter,AvailabilityZoneFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,PciPassthroughFilter | 增加 PciPassthroughFilter |

####  

#### 2.2.2 Nova 步骤

（1）创建一个 nova flavor 并设置属性。本例子中直接设置已有 m1.small 的属性：

nova flavor-key m1.small set "pci\_passthrough:alias"="a1:1"   


其中，

* "pci\_passthrough:alias"  是固定的 key 字符串，用于指定该 flavor 对 PCI 设备的需求

* "a1:1" 的格式是 'alias\_name\_x:count, alias\_name\_y:count, ... ' ，可以指定多个，每个 alias\_name 由 pci\_alias 配置项中的 ”name“ 指定。

（2）使用该 flavor 创建一个虚机

nova boot --image new1 --key\_name test --flavor m1.small 123  


（3）待虚机变为可用状态后，登录，使用 lspci 命令查看 GPU 设备 

nova ssh --private 123 -i test.pem  


###  

### 2.3 Nova PCI 相关代码分析

####  

#### 2.3.1 Nova 资源申请过滤、资源申请和状态周期性汇报流程

![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYyJYOZLQUn5xuIVicKQ0iamlElpDHmGPl4IEVnD2OuWfZkcCcGu47k1nlA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


A： KVM 计算节点上的 PCI 设备状态的获取

nova-compute 的 resource\_tracker 周期性地获取 KVM 计算节点的 PCI 状态并写进数据库：

2016-02-03 03:20:58.191 INFO nova.compute.resource\_tracker \[req-5da1fbd0-3f97-4a88-83be-51f938b60860 None None\] Final resource view: name=hkg02kvm002ccz023 phys\_ram=32204MB used\_ram=25088MB phys\_disk=314GB used\_disk=240GB total\_vcpus=24 used\_vcpus=4 pci\_stats=&lt;nova.pci.stats.PciDeviceStats object at 0x7f4e60d32910&gt;  


类 nova.pci.stats.PciDeviceStats 的实现在 /nova/pci/stats.py 文件中。其数据格式为：

\| \[{"count": 5, "vendor\_id": "8086", "product\_id": "1520", "extra\_info":'{}'}\],  


也就是每一种由配置项 pci\_passthrough\_whitelist  所指定的可分配给虚机的 PCI 设备的可用数目。

  


B: Nova scheduler 使用 DB 中的 PCI 设备状态数据来过滤出能满足请求所要求的PCI资源的计算节点

用户使用某个配置了 PCI 设备需求的 Nova flavor 来创建虚机

-&gt; nova-api 读取所使用的 flavor 的 ”pci\_passthrough:alias“ 属性的值，该值指定了所使用的 PCI 设备的属性和数量，其数据形式为 ”\| \[{"count": 1, "vendor\_id": "8086", "product\_id": "1520",}\].“

-&gt; nova-scheduler 的 PciPassthroughFilter 匹配每个 KVM host 保存在数据库中的 pci\_stats 和该 request 所要求的 PCI 资源，来确定每个 host 能不能满足虚机所要求的 PCI 设备的需求。如果不能满足，则返回 false；能满足则返回 true，表明该 host 满足了该 filter 的要求。

  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYySNpxLOngPz0A0lF2uh0h40Zb6l0zvb5X2QV86yPs3M6ibpic0BkWkCbA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

def host\_passes\(self, host\_state, spec\_obj\):        """Return true if the host has the required PCI devices."""  
       pci\_requests = spec\_obj.pci\_requests        if not pci\_requests:            return True        if not host\_state.pci\_stats.support\_requests\(pci\_requests.requests\):  
           LOG.debug\("%\(host\_state\)s doesn't have the required PCI devices"  
" \(%\(requests\)s\)",  
                     {'host\_state': host\_state, 'requests': pci\_requests}\)            return False        return True  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYySNpxLOngPz0A0lF2uh0h40Zb6l0zvb5X2QV86yPs3M6ibpic0BkWkCbA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

  


C: PCI 设备管理和分配

  Nova 使用类 ResourceTracker 来统一管理计算节点上的所有资源，包括资源发现（discover）、声明（claim）、分配（allocate）和释放（free）等操作。PCI 设备也是受管理资源的一种。类似其它资源，PCI 资源的信息也是永久保存在数据库中，这也方便使用者来查询这些信息。PCI 设备包括如下几种状态：available/claimed/allocated/deleted/removed。

  


PCI 资源分配的基本步骤：

（a）创建虚机：调用 def \_build\_instance\(self, context, request\_spec, filter\_properties, requested\_networks, injected\_files, admin\_password, is\_first\_time,node, instance, image\_meta, legacy\_bdm\_in\_spec\) 方法

  


（b）claim PCI 资源：调用 resourceTracker.instance\_claim\(context, instance, limits\) 方法来 claim 资源，包括内存、磁盘、numa 和 PCI 设备等。如果 claim 失败，则支持跑出错误。

  


（c）分配 PCI 资源：调用 self.\_update\_usage\_from\_instance\(context, self.compute\_node, instance\_ref\)  来将资源标记为占用。该函数中，会调用 self.pci\_tracker.update\_pci\_for\_instance\(context, instance\) 来将 PCI 设备的状态保存到 pci\_tracker。

####  

#### 2.3.2 虚机创建

（1）将 pci resource manager 分配好的 PCI 设备后加入到 guest中：

if virt\_type in \('xen', 'qemu', 'kvm'\):            for pci\_dev in pci\_manager.get\_instance\_pci\_devs\(instance\):  
               guest.add\_device\(self.\_get\_guest\_pci\_device\(pci\_dev\)\)  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYySNpxLOngPz0A0lF2uh0h40Zb6l0zvb5X2QV86yPs3M6ibpic0BkWkCbA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

def \_get\_guest\_pci\_device\(self, pci\_device\):  
  
       dbsf = pci\_utils.parse\_address\(pci\_device\['address'\]\)  
       dev = vconfig.LibvirtConfigGuestHostdevPCI\(\)  
       dev.domain, dev.bus, dev.slot, dev.function = dbsf  
  
       \# only kvm support managed mode        if CONF.libvirt.virt\_type in \('xen', 'parallels',\):  
           dev.managed = 'no'  
if CONF.libvirt.virt\_type in \('kvm', 'qemu'\):  
           dev.managed = 'yes'  
  
return dev  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYySNpxLOngPz0A0lF2uh0h40Zb6l0zvb5X2QV86yPs3M6ibpic0BkWkCbA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

然后再调用 xml = conf.to\_xml\(\) 方法得到 guest 的 libvirt xml。一个分配的 PCI 设备的 Libvirt XML 定义示例如下：

![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYySNpxLOngPz0A0lF2uh0h40Zb6l0zvb5X2QV86yPs3M6ibpic0BkWkCbA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

&lt;hostdev mode='subsystem' type='pci' managed='yes'&gt;  
&lt;driver name='vfio'/&gt;  
&lt;source&gt;  
&lt;address domain='0x0000' bus='0x86' slot='0x00' function='0x0'/&gt;  
&lt;/source&gt;  
&lt;alias name='hostdev0'/&gt;  
&lt;address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/&gt;  
&lt;/hostdev&gt;  


![](https://mmbiz.qpic.cn/mmbiz/TkGS5WgibziaqRP2rySaAK2CVdv79V3mYySNpxLOngPz0A0lF2uh0h40Zb6l0zvb5X2QV86yPs3M6ibpic0BkWkCbA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

  


（2）虚机完成启动后，登录它，再使用 lspci 就可以看到该透传的 GPU 了 

这是 Cirros 客户机中的输出：

$ lspci -k  
...00:05.0 Class 0100: 1af4:1001 virtio-pci00:06.0 Class 0300: 10de:11bf00:07.0 Class 00ff: 1af4:1002 virtio-pci  


这是 Ubuntu 客户机中的输出：

ubuntu@ubuntu-gpu:~$ lspci -nn  
...  
00:06.0 VGA compatible controller \[0300\]: NVIDIA Corporation GK104GL \[GRID K2\] \[10de:11bf\] \(rev a1\)



（3）在主机上查看两个 GPU 的状态，可以看出已分配和未分配的状态是不同的

stack@hkg02kvm002ccz023:~/logs$ readlink /sys/bus/pci/devices/0000\:85\:00.0/driver \#这是已经分配给虚机的../../../../../../bus/pci/drivers/pci-stubstack@hkg02kvm002ccz023:~/logs$ readlink /sys/bus/pci/devices/0000\:86\:00.0/driver \#这是未分配给虚机的../../../../../../bus/pci/drivers/vfio-pci

