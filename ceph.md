Ceph iSCSI Gateway Demo安装配置（上）

1

背景介绍

ceph 作为一个真正意义上的统一存储系统，有着很好的应用前景，但到目前为止有多 种原因限制了其在传统存储应用领域真正大范围的应用，如客户端仅支持 GNU/Linux 系统， 内核态客户端实现也仅会合入高版本的内核中等。而对于 iSCSI 这种传统的存储应用而言， 由于客户端配置简单且足够通用，常见的各种系统（包括操作系统和应用系统）一般都对 iSCSI 有很好的支持，因此为了扩大 ceph 的应用范围，特别是应对只支持 iSCSI 的系统， ceph 必须通过某种途径实现对 iSCSI 的支持。

  


2

方案选型

ceph 集群目前支持三种形式的存储接口：文件、对象、块，其中块接口\(即 RBD\)与 SCSI 块设备读写所要求的接口一致，因此可以基于 ceph 的 RBD 提供 SCSI 存储系统后端。基于 RBD 的 iSCSI 支持有多种实现思路，根据我们团队的现状，选择以下方案： 选择合适的 iSCSI target 为其增加 RBD 后端支持，并在终端用户与 ceph 集群之 间架设 iSCSI 网关（可以扩展成其它类型的如 FC 网关）。  


基本的架构图如下：

![](https://mmbiz.qpic.cn/mmbiz/3zoYydJjr12uGQmQcqPthicag8B3tlNXNr4A2CELN2pKDZhw3UJ407PGS01brGQCxdaqkGoMyAj88a8sOibrgn5Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





    图1. Ceph iSCSI Gateway基本框架  


  


从上图可以看出，Ceph iSCSI Gateway基本框架包括三个部分，ceph cluster、iscsi gateway和client。下面简单介绍以下各个组件的方案选择。  
  


## **2.1 SCSI target 框架**

GNU/Linux 系统中流行的 SCSI target 框架或 iSCSI target 实现有如下几种： tgt、SCST、 Linux-IO。 

**1. tgt**

tgt 是一个用户态的 SCSI target 框架，在 GNU/Linux 内核直接集成 SCSI target框架之前，这是一个绝对主流的框架。

它的优点是：

1）简单，方便使用和维护；

2）另外已经有ceph的target driver，只是需要做性能优化；

3）因为工作在用户态，所以即使挂掉了，也不会对其他运行的程序产生影响；

它的缺点是：

1）支持的传输协议较少；

2）对SCSI协议支持比较简单，一些cluster中的特性比如PR等都不支持，所以基于stgt的方案不能在cluster中使用；

3）由于是用户态框架，性能问题较差，根据网上的相关数据， tgt 在使用本地存储的情况下，性能相比后面会提到的 SCST、 LIO 等是有一定差距的。



**2.SCST**

SCST的核心模块工作在内核里，可以支持通过系统模块（VFS、块层）访问的后端存储如块设备、文件设备以及 passthrough的scsi设备。

它的优点是：

1）支持更多传输协议

2）针对性能做了特殊的优化

3）除了基本的SCSI协议支持外，还有一些高级支持：

SCST支持永久性预留（Persistent Reservation, PR）；这是一个用于高可用集群中的存储设备的 I/O 隔离与存储设备故障切换、接管的特性。通过使用 PR 命令，initiator 可以在一个 target 上建立、抢占、查询、重置预留策略。在故障接管过程中，新的虚拟资源可以重置老的虚拟资源的预留策略，从而让故障切换更快、更容易地进行。

SCST 可以使用异步事件通知（AEN）来通告会话状态的变更。AEN 是一个 SCSI target 用来向 initiator 进行 target 端的事件告知的协议特性，即使在没有服务请求的时候也可以进行。于是 initiator 就可以在 target 端发生事件时，如设备插入、移除、调整尺寸或更换介质时，可以得到通知。这让 initiator 可以以即插即用的方式看到 target 的变化。

4）SCST 的开发者声称，它们的设计在健壮性和安全性方面更加符合 SCSI 标准。SCSI 协议要求，如果一个 initiator 要清除另一个 initiator 的预留资源时，预留者必须要得到清除通知，否则，多个 initiator 都可能来改变预留数据，就可能会破坏数据。SCST 可以实现安全的预留、释放操作，避免类似事情发生。

5）SCST 也支持非对称逻辑卷分配（ALUA）。ALUA 允许 target 管理员来管理 target 的访问状态和路径属性。这让多路径路由机制可以选择最好的路径，从而根据 target 的访问状态，优化带宽的使用。换句话说，在多路径环境下，target 管理员可以通过改变访问状态来调整 initiator 的路径。

6）各大存储服务提供商都是基于SCST。

7）提供更细粒度的访问控制策略以及QoS保证机制（限制initiator连接的个数）。

它的缺点是：

1）结构复杂，二次开发成本较高。

2）工作在kernel，如果挂了，会导致整个机器down掉，影响其他程序。

3）kernel部分没有并入linux，需要手工编译。



**3. LIO**

LIO 也即 Linux-IO，是目前 GNU/Linux 内核自带的 SCSI target 框架（自 2.6.38版本开始引入，真正支持 iSCSI 需要到 3.1 版本） ，对 iSCSI RFC 规范的支持非常好，包括完整的错误恢复都有支持。整个 LIO 是纯内核态实现的，包括前端接入和后端存储模块，为了支持用户态后端，从内核 3.17 开始引入用户态后端支持，即 TCMU\(Target Core Module in Userspace\)

优点：

1）支持较多传输协议

2）代码并入linux内核，减少了手动编译内核的麻烦。

3）提供了python版本的编程接口rtslib。

4）LIO在不断backport SCST的功能到linux内核，社区的力量是强大的。

5）LIO也支持一些SCST没有的功能。如LIO 还支持“会话多连接”（MC/S）。

6）LIO支持最高级别的ERL。

缺点：

1）不支持AEN，所以target状态发生变化时，只能通过IO或者用户手动触发以检测处理变化。

2）结构相对复杂，二次开发成本较高。

3）工作在内核态，出现问题会影响其他程序的运行。



**4. 总结**

总的来说，stgt构建一个规模不是很大的iSCSI target。

LIO支持FC、SRP等协议，但性能、稳定性不是很好。

SCST适合构建一个企业级的高性能、高稳定性的存储方案，各大存储服务提供商都是基于SCST。因此我们选择了SCST。  
  


**2.2 多路径**

普通的电脑主机都是一个硬盘挂接到一个总线上，这里是一对一的关系。而到了有光纤组成的SAN环境，或者由iSCSI组成的IPSAN环境，由 于主机和存储通过了光纤交换机或者多块网卡及IP来连接，这样的话，就构成了多对多的关系。也就是说，主机到存储可以有多条路径可以选择。主机到存储之间 的IO由多条路径可以选择。每个主机到所对应的存储可以经过几条不同的路径，如果是同时使用的话，I/O流量如何分配？其中一条路径坏掉了，如何处理？还 有在操作系统的角度来看，每条路径，操作系统会认为是一个实际存在的物理盘，但实际上只是通向同一个物理盘的不同路径而已，这样是在使用的时候，就给用户 带来了困惑。多路径软件就是为了解决上面的问题应运而生的。

多路径的主要功能就是和存储设备一起配合实现如下功能：

1.故障的切换和恢复

2.IO流量的负载均衡

3.磁盘的虚拟化



由于多路径软件是需要和存储在一起配合使用的，不同的厂商基于不同的操作系统，都提供了不同的版本。并且有的厂商，软件和硬件也不是一起卖的，如果要使用多路径软件的话，可能还需要向厂商购买 license 才行。 业界比较常见的多路径功能软件有 EMC 的PowerPath， IBM 的 SDD，日立的 Hitachi Dynamic Link Manager 和广泛使用的 linux 开源软件 multipath和windows server的MPIO。

由于我们的Demo是把多路径接入ceph，把ceph通过scsi target软件映射到主机上使用，因此我们多路径软件选择的是linux 开源软件通用多路径 multipath和windows server的MPIO。

  


3

总体概述



Ceph iSCSI Gateway Demo整体框架包括三个部分，ceph cluster、iscsi gateway和client。

iscsi gateway可以部署多个，每个gateway包括iscsi target和内核rbd，iscsi target使用了开源软件SCST，块存储由ceph的内核模块rbd提供接口。每一个存储资源可以映射到多个gateway上，通过任何一个gateway都可以访问到数据。将一个image映射到gateway本地后，再把这个块设备配置为target的后端存储。

Client（linux）包括multpath和iscsi initiator，initiator使用open-iscsi，每个initiator可以连接到多个Iscsi target。对于多个target后端对应的同一个image，则使用多路径软件聚合成一个块设备后提供给用户使用。这样可以解决网络故障或者gateway故障后，通过另一条链路能够继续提供服务。

![](https://mmbiz.qpic.cn/mmbiz/3zoYydJjr12uGQmQcqPthicag8B3tlNXN857AK8q7cPJPRVzrXwY4iaWWvMKRLggXv1pPG42MaBhrcxv2m6zJc4w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



  图2  Ceph iSCSI Gateway Demo架构  


  


4

Ceph iSCSI Gateway Demo 安装

##  

## **4.1 iSCSI Gateway**

## 

**4.1.1下载源码**

SCST的版本选择的是3.0.1，这个版本是社区里最新的稳定版。SCST的安装需要重新编译源码，相关的源码下载地址如下

1、scst源码地址：

http://sourceforge.net/projects/scst/files/scst/scst-3.0.1.tar.bz2/download

2. scstadmin源码地址：http://sourceforge.net/projects/scst/files/scstadmin/scstadmin-3.0.1.tar.bz2/download

3. iscsi-scst源码地址：http://sourceforge.net/projects/scst/files/iscsi-scst/iscsi-scst-3.0.1.tar.bz2/download  
  


**4.1.2安装SCST**

拷贝scst-3.0.1.tar.bz2到iscsi gateway上，执行以下命令： 

tar   -jxvf    scst-3.0.1.tar.bz2

cd  scst-3.0.1

make

make install



**4.1.3安装iscsi-scst**

拷贝iscsi-scst-3.0.1.tar.bz2到iscsi gateway上，执行以下命令：

tar   -jxvf   iscsi-scst-3.0.1.tar.bz2

cd  iscsi-scst-3.0.1

make

make install



**4.1.4安装scstadmin**

拷贝scstadmin-3.0.1.tar.bz2到iscsi gateway节点上，执行以下命令：

 tar  -jxvf  scstadmin-3.0.1.tar.bz2

cd  scstadmin-3.0.1

make

make install



**4.1.5启动iscst target**

执行下面的命令加载scst内核模块

modprobe scst

modprobe scst\_vdisk

modprobe iscsi-scst

执行如下命令启动scst的用户态进程

/etc/init.d/scst start

/usr/local/sbin/iscsi-scstd



**4.2. client（iscsi initiator）安装**

**4.2.1 linux的client端安装**

我们的平台的操作系统是ubuntu，因此安装时直接用apt-get即可

4.2.1.1安装open-iscsi

在client（iscsi initiator）所在的节点上安装open-iscsi：

apt-get install open-iscsi

4.2.1.2 多路径软件安装

在client（iscsi initiator）所在的节点上安装multipath：

apt-get install multipath-tools



**4.2.2 windows的client端安装（后续补充）**

注意：由于Windows 只有Server 才包括 MPIO 软件包，因此我们的demo环境windows操作系统选择的是Windows Server 2008。

