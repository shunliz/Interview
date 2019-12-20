云平台的弹性伸缩，大致可以分为横向伸缩（scalein/out）和纵向伸缩（scale up/down）。纵向伸缩是修改原有云服务器的配置，包括磁盘的大小、CPU 的运算能力、内存的大小以及网卡和 IP 的流量限制等等。传统机器要升级配置，往往是需要停机的；在虚拟化环境下，实现在线升级配置从技术上和操作上都要更为容易一些，从而让客户在服务器负载超出预估需要升级配置时，依然能够保证服务的可用性。华云云主机产品将结合QEMU内存和CPU热插拔的技术，实现生产环境下不中断业务的在线系统升级。

本文将对 QEMU 的虚拟机内存热插拔功能的使用做一个简单的介绍。QEMU 在 2.1 中引入了memoryhot-plug 支持，在 2.4 中引入了 memoryhot-unplug 支持，Libvirt 中相应的支持在 1.2.14 开始引入，本文主要介绍 hot-plug。

  


环境准备

  


这里以 CentOS 7 作为宿主机环境，目前自带源中的 QEMU 版本为 1.5.3，Libvirt 版本为 1.2.8，需要进行升级。

  


升级 QEMU

  


可以使用 RedHat 提供的 SRPM 包升级到 2.1.2。执行以下命令构建 RPM包：

**清单 1. 构建 qemu-kvm RPM 包**

1.    yum -y install rpm-build

2.    wget http://ftp.redhat.com/pub/redhat/linux/enterprise/7Server/en/RHEV/SRPMS/qemu-kvm-rhev-2.1.2-23.el7\_1.9.src.rpm

3.    rpm -ivh qemu-kvm-rhev-2.1.2-23.el7\_1.9.src.rpm

4.   yum -y groupinstall "Development Tools"

5.    yum -y install zlib-devel libaio-devel libcurl-devellibssh2-devel libseccomp-devel lzo-devel snappy-devel numactl-devel pixman-devellibcap-devel brlapi-devel check-devel librdmacm-devel libpng-devel libjpeg-develnss-devel libuuid-devel bluez-libs-devel glusterfs-devel glusterfs-api-devellibrbd1-devel librados2-devel SDL-devel gnutls-devel pciutils-devel pulseaudio-libs-devellibiscsi-devel ncurses-devel libattr-devel libusbx-devel usbredir-devel spice-server-develsystemtap-sdt-devel cyrus-sasl-devel texi2html texinfo iasl

6.    rpmbuild -bb ~/rpmbuild/SPECS/qemu-kvm.spec

构建完毕后安装 RPM 包：

**清单 2. 安装 qemu-kvm**

1.    yum -y install ipxe-roms-qemu seabios-bin seavgabios-binsgabios-bin

2.    rpm -ivh ~/rpmbuild/RPMS/x86\_64/qemu-\*

  


升级 Libvirt

  


这里使用 CBS 提供的 libvirt 包，首先加入 CBS 源，创建文件/etc/yum.repos.d/cbs.repo，内容如下：

**清单 3. CBS libvirt repo配置**

1.    name=virt7-xen-46-testing

2.    baseurl=http://cbs.centos.org/repos/virt7-xen-46-testing/x86\_64/os/

3.    enabled=1

4.    gpgcheck=0

然后安装 libvirt：

**清单 4. 安装 libvirt**

1.    yum -y install libvirt

  


使用 memory hot-plug

  


一般情形下，QEMU 中的虚拟机都是通过 Libvirt 创建的，要支持内存热插拔，在创建虚拟机时需指定**maxMemory**配置，并且需指定至少一个 NUMA 节点（虚拟机中的NUMA 节点，非宿主机中的）：

**清单 5. 虚拟机 maxMemory 和 numa 配置**

1.    &lt;maxMemory slots='32' unit='KiB'&gt;68719476736&lt;/maxMemory&gt;

2.    ...

3.    &lt;cpu mode='host-model'&gt;

4.    &lt;model fallback='allow'/&gt;

5.    &lt;numa&gt;

6.    &lt;cell id='0' cpus='0' memory='4194304' unit='KiB'/&gt;

7.    &lt;/numa&gt;

8.    &lt;/cpu&gt;

在清单 5 中，&lt;maxMemory&gt;内的内存值（这里是 64 GiB）表示通过 hot-plug 可以达到的内存的上限（包含虚拟机初始内存）。其中 slots 表示 DIMM 插槽的数量，每个插槽在运行时都可以插入一个内存设备，上限是 255 个。而&lt;numa&gt;内的配置用于指定虚拟机内的 NUMA 拓扑，清单 5 的配置中虚拟机内只有一个 NUMA 节点。

按照以上要求创建的虚拟机，可以运行时插入内存设备，具体操作时需要先创建一个设备信息描述文件：

**清单 6. 内存设备配置格式**

1.    &lt;memory model='dimm'&gt;

2.    &lt;target&gt;

3.    &lt;size unit='KiB'&gt;524287&lt;/size&gt;

4.    &lt;node&gt;0&lt;/node&gt;

5.    &lt;/target&gt;

6.    &lt;/memory&gt;

清单 6 中的&lt;size&gt;部分指定设备的内存容量（这里是 512MiB），&lt;node&gt;部分指定插入到虚拟机的哪个 NUMA 节点。

然后使用attach-device命令挂载设备（假设上述的设备信息文件名为memdev.xml，虚拟机名字为testdom）：

**清单 7. 挂载设备**

1.    virsh attach-device --live testdom memdev.xml

最后，根据虚拟机内操作系统的不同，需要在其中进行一些操作激活新插入的内存，下面将针对 Linux 和 Windows 分别进行叙述。

  


Linux 虚拟机

  


Linux 中要完整的支持 memoryhot-plug，需要较新的内核。这里以 CentOS 7 作为示例，测试虚拟机的内存为 1GiB，有 8 个内存块：

图 1. 插入内存前的内存块  
![](https://mmbiz.qpic.cn/mmbiz_png/3zoYydJjr10JExP5ECjKaKxIkzU6cCwllUjMAxSXuNyc4t2icXib6EdnH6grJFyqz6HbiabUMrMTD0kicQ1HUBwEEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
图 2. 插入内存前的总内存  


![](https://mmbiz.qpic.cn/mmbiz_png/3zoYydJjr10JExP5ECjKaKxIkzU6cCwl9hkHriaWwBlJ78UoI0QnUVbbelEw2dKnGUGdD08XwSP5roEMeyIlB7A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


使用attach-device插入新内存以后，可以看到多了 4 个内存块：

图 3. 新增的内存块  
![](https://mmbiz.qpic.cn/mmbiz_png/3zoYydJjr10JExP5ECjKaKxIkzU6cCwlQ6GuvqHIRyDzS7geBdF6EBbcU7WezK86fjJxISCpPUQQNUrSEicSRLQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


插入的内存需要手动激活，激活后就可以看到系统总内存已经改变了：

图 4. 激活内存和总内存变化  
![](https://mmbiz.qpic.cn/mmbiz_png/3zoYydJjr10JExP5ECjKaKxIkzU6cCwlTniaTjiammmhELHm5xsbSTN0Uj0whdnE5ib1z0TKKeruv7ZZSRk783oEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


为了让系统自动 online 添加的内存，可以设置 udev 规则，创建/etc/udev/rules.d/99-hotplug-memory.rules文件，内容如下：

**清单 8. 自动 online 内存的 udev 规则**

1.    \# automatically online hot-plugged memory

2.    ACTION=="add", SUBSYSTEM=="memory",ATTR{state}="online"

  
文件保存以后会立刻生效。

在 CentOS 6 中，由于编译内核时已经开启配置，插入的内存会被自动 online；不过如果内核的版本不够高，重启虚拟机以后系统将识别不到内存设备。升级到 2.6.32内核的最新版本可以避免此问题，又或者可以采用手动 probe 内存块的方法（见参考资料部分的说明），但是要注意内存起始地址不能错误，否则 online 的时候系统会奔溃。

  


Windows 虚拟机

  


本文中测试的 Windows 版本为 Windows 2008 R2 Datacenter x64，使用attach-device命令挂载内存设备以后，新加入的内存会自动激活，不需要额外的操作，图 4 和 图 5 分别包含了挂载前和挂载后的总内存信息（看物理内存总数这一项）。可以观察到插入内存设备时 CPU 使用率有一瞬间的增长，之后会回落下去。

图 5. 插入内存设备前  
![](https://mmbiz.qpic.cn/mmbiz_png/3zoYydJjr10JExP5ECjKaKxIkzU6cCwlBBCPiaKlb4ddLfZG40KO2gGDnVzEsdgUcUu5zM2oQh5O1RJPgShsATw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


图 6. 插入内存设备后

![](https://mmbiz.qpic.cn/mmbiz_png/3zoYydJjr10JExP5ECjKaKxIkzU6cCwl9h1zvED6NB8Mr7Low5zbF8e9cib6nq8d16upDHk7YTb9MWdhjJhTicEQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


**目前支持内存热插的 Windows 版本有：**

Windows Server 2008 R2,     Enterprise Edition and Datacenter Edition

Windows Server 2008, Enterprise     Edition and Datacenter Edition

Windows Server 2003, Enterprise     Edition and Datacenter Edition

所有 Windows 系统都不支持内存热拔操作。

参考资料

qemu  memory-hotplug.txtQEMU 关于 memory  hot-plug 的说明。

Domain XML Format虚拟机配置的&lt;maxMemory&gt;和&lt;memory&gt;可以看这里。

linux  memory-hotplug.txtLinux 中内存热插拔支持的信息，关于内存块 probe 的部分可以看第 4 小节的说明。

Operating system support for hot-add memory in Windows Server官方的 Windows 热插拔版本支持说明。

vSphere  Memory Hot Add/CPU Hot Plug这里有一张不同版本 Windows 对于 CPU 和内存热插拔支持的表格。

vSphere  5.1: Hot add RAM and CPU这里有另外一张不同版本     Windows 对于 CPU 和内存热插拔支持的表格。

Decreased performance, driver load failures, or system instability may occur on Hot Add Memory systems that are running Windows Server 2003WindowsServer 2003 使用内存热插的问题。

\[Qemu-devel\] \[PATCH 00/35\] pc: ACPI memory hotplug社区关于 memory hot-plug 的信息。

  


小结

  


本文简单介绍了 QEMU 中的 memory hot-plug 功能，memory hot-unplug 由于操作系统本身支持的不完善，并没有予以介绍。这个功能配合内存监控可以做到自动根据负载增加内存，对于内存消耗型的应用（比如缓存）还是很有用的，华云数据即将在云主机产品中支持内存和CPU热插拔的功能，实现资源纵向弹性伸缩，请大家持续关注我们的产品动态。

