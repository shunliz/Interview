可信计算技术是实现系统安全的重要手段。为了保证虚拟机运行环境的完整性和安全性，本文介绍了一种KVM虚拟机平台上的可信计算架构实现，解决虚拟机平台的可信认证问题，使得信任链能够延伸到客户操作系统，为虚拟机的安全提供了保障。

  


一、可信计算简介

可信计算（Trusted Computing）是由可信计算组（Trusted Computing Group）制定的一项技术。其核心思想是在可信计算机系统中引入一个可信的TPM（Trusted Platform Module）芯片，通过对TPM的度量建立一个逐级认证的信任链，最终将这种信任扩大到整个计算机系统，以提高系统整体的安全性。

  


可信计算平台的核心是可信计算平台模块\(TPM\)，它通常具有密码运算能力和存储能力，内部具有受保护的安全存储单元，是一个含有密码运算部件和存储部件的小型片上系统。作为可信平台的构件，TPM是整个可信计算平台的基石，TPM向操作系统、TSS等提供诸多安全方面的支持，通过TPM的功能支持，可信计算平台能够提供多种安全服务。

  


可信计算的基本思想是，首先建立一个可信根，由可信根开始建立一条信任链，从可信根到可信硬件平台，再到操作系统，最后到应用程序，逐级验证，把这种信任关系传递到整个计算机系统中，从而保证整个计算机体系的可信性。可信计算平台重要的基础部件包括可信平台模块TPM和TCG软件协议栈TSS\(TCG Software Stack\)。可信软件协议栈处于TPM之上，应用软件之下，是对可信计算平台提供支持的软件，它的设计目标是对使用TPM功能的应该程序提供一个唯一入口，并提供对TPM的同步访问管理。TSS平台软件从结构上可以分为四层，自下至上分别是TPM驱动程序库\(TPM Device Driver Library, TDDL\)、可信软件栈核心服务\(TSS Core Services, TCS\)和可信服务提供者\(Trusted Service Provider, TSP\)。

  


TSS的体系结构如图所示：

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLMPqzlO04zAw6XYveGWNgZVArwtOKT9s2ClBuOMADjUibLdKGqxR1mbjNKStz9gmWrX9JoaG4WMh5A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


  


二、实现方案介绍

  


由前面的介绍可以看到在虚拟机上实现可信计算功能的关键是在虚拟机中实现TPM设备的模拟。传统的可信计算技术是每台物理计算机带有一块TPM芯片，通过TPM芯片的安全性功能保证系统的安全；TPM1.2规范中没有考虑虚拟化的场景，因此不支持多个虚拟机同时使用一块TPM芯片；QEMU/KVM目前实现的做法是将TPM芯片透传给虚拟机，这使得在一台物理机上只有一个虚拟机可以使用TPM设备，且物理计算机与虚拟机不能同时使用TPM芯片，限制了可信计算功能在虚拟机上的应用。为实现多个虚拟机同时使用TPM功能，方案通过软件实现来模拟虚拟机上的TPM芯片功能，称为虚拟TPM设备。虚拟TPM设备的功能与接口与物理TPM芯片保持一致，虚拟机的TPM驱动和内核看不到设备差异，不知道使用的是虚拟设备。信任链从物理机的TPM芯片通过物理机驱动、虚拟化层软件一直延伸到客户操作系统中。

  


系统架构如图所示：

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLMPqzlO04zAw6XYveGWNgZVRtkAvSwxwsaOZxDibbqdBok3Kic2SVtD5ibUYUhIibz2JgzlZr7oIyV6uA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  


  


系统中各部分组件介绍如下：

1

**libtpms**

libtpms是一个库，用于向QEMU中提供TPM功能。libtpms提供了规范的公用API接口，实现了以下TPM功能的模拟，对称/非对称加密、安全存储、完整性度量和安全签名等。

libtpms的编译命令如下：

\# ./bootstrap.sh

\# ./configure --prefix=/usr

\# make

\# sudo make install

  


2

**QEMU部分**

QEMU目前只支持将物理的TPM芯片透传给虚拟机使用，一台物理机上只能有一个虚拟机可以使用TPM芯片；要实现使用libtpms库，通过软件模拟的方式向虚拟机提供TPM功能，需要修改软件流程打补丁。QEMU补丁主要实现的功能是新增一种TPM设备类型和TPM设备的后端驱动。TPM设备的后端驱动接收虚拟机发送过来的TPM设备操作，解析操作类型并调用libtpms相应的接口完成TPM设备操作，并将操作结果返回给虚拟机。

打过补丁后，qemu针对tpm设备的参数格式如下：

TPM device options:

-tpmdev passthrough,id=id\[,path=path\]\[,cancel-path=path\]

                use path to provide path to a character device; default is /dev/tpm0

                use cancel-path to provide path to TPM's cancel sysfs entry; if

                not provided it will be searched for in /sys/class/misc/tpm?/device

-tpmdev libtpms,id=id,nvram=drive-id

                use nvram to provide the NVRAM drive id

  


3

**seabios**

seabios 是 x86 结构下的一种 BIOS 的开源实现，可以完成初始化硬件工作，实现一些启动逻辑。

  


支持TPM的seabios编译方法如下：

  


下载最新的seabios源码，解压。

  


在seabios目录下，执行make menuconfig。

  


进入BIOS interfaces后选择TPM support and TCG extensions

  


然后保存，退出。

  


执行make，编译完成后seabios的执行文件为out/bios.bin，使用这个文件作为QEMU虚拟机启动时使用的bios就可以。

  


三、虚拟机运行

libtpms需要一个模拟的nvram设备，因此需要先创建相应的设备模拟文件。设备创建命令如下：

qemu-img create -f qcow2 nvram.img 1M

启动多个虚拟机，相应的命令如下：

/home/x86\_64-softmmu/qemu-system-x86\_64 -name instance-00000198 -machine pc-i440fx-1.7,accel=kvm,usb=off -cpu SandyBridge,+pdpe1gb,+osxsave,+dca,+pcid,+pdcm,+xtpr,+tm2,+est,+smx,+vmx,+ds\_cpl,+monitor,+dtes64,+pbe,+tm,+ht,+ss,+acpi,+ds -bios /home/seabios-1.9.1/out/bios.bin -m 5120 -realtime mlock=off -smp 4,sockets=4,cores=1,threads=1 -uuid f91e1fb8-04aa-4e31-9bf8-ac1050d127fa -no-user-config -nodefaults -rtc base=utc,driftfix=slew -global kvm-pit.lost\_tick\_policy=discard -no-hpet -no-shutdown -boot order=c,menu=on,splash-time=5000,strict=on -device piix3-usb-uhci,id=usb,bus=pci.0,addr=0x1.0x2 -drive file=/home/centos.img,if=none,id=drive-virtio-disk0,format=qcow2,cache=none -device virtio-blk-pci,scsi=off,bus=pci.0,addr=0x3,drive=drive-virtio-disk0,id=virtio-disk0 -chardev pty,id=charserial0 -device isa-serial,chardev=charserial0,id=serial0 -device usb-tablet,id=input0 -vnc 0.0.0.0:0 -k en-us -device cirrus-vga,id=video0,bus=pci.0,addr=0x2 -device virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x4 -msg timestamp=on -drive file=/home/nvram.img,if=none,id=nvram0-0-0,format=qcow2 -device tpm-tis,tpmdev=tpm-tpm0,id=tpm0 -tpmdev libtpms,id=tpm-tpm0,nvram=nvram0-0-0

使用的虚拟机内核版本为3.10.0，包含了IMA和TPM驱动相关的模块，否则需要单独编译IMA和TPM驱动模块，并将模块插入内核。

虚拟机启动后在虚拟机中可以看见TPM设备，上电过程有tpm\_tis模块的芯片ID的打印信息，表示QEMU已经能够正确模拟TPM设备。

  


在虚拟机中安装TrouSerS、Tpm-tools、TSS API testsuite等软件。

  


TrouSerS是一个开源TSS软件栈，兼容目前的TSS 1.1以及1.2规范，能够为上层用户应用提供访问TPM的接口。Tpm-tools是一个开源的工具，提供了TPM常用功能的使用范例。TSS API testsuite是一个开源的工具，提供了TSS API的测试功能。运行Tpm-tools、TSS API testsuite等测试工具，测试结果表明虚拟机中TPM功能正常运行。

