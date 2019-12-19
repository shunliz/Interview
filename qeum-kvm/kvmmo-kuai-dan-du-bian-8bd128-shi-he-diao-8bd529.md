在说kvm模块单独编译之前，难免设计到linux内核模板的编写，所以这里也稍微提一下。



1

linux内核模块环境搭建



这里有2种方法：

**1.1 升级内核：**

升级当前系统的kernel，具体编译步骤这里不再详细说明，简单表述一下：如果想在当前的linux系统上面，不用修改配置文件来编译内核，就将/boot/config-\*\*\*文件拷贝至/home/pizhi/linux-4.6.4/.config



> \[root@pizhi-kernel boot\]\# pwd
>
> /boot
>
> \[root@pizhi-kernel boot\]\# cp config-3.10.0-229.el7.x86\_64 /home/pizhi/linux-4.6.4/.config



  


**注意：执行完上面的cp命令以后，仍然需要使用make menuconfig命令并保存配置。**

编译之前需要安装rpmbuild工具：



> make rpm



  


这一步会在/root/rpmbuild/RPMS生成对应的kernel rpm包。

更新：



> yum install ./\*.rpm



### **1.2 安装kernel-devel包**

  


不需要像1.1中的花很长时间升级内核，只需要安装kernel-devel rpm包即可。

安装：



> yum install kernel-devel



  


如果源没啥问题，基本安装的和kernel rpm包的版本一致即可。

测试：



> hello.c：
>
> \#include &lt;linux/init.h&gt;
>
> \#include &lt;linux/module.h&gt;
>
> static int hello\_init\(void\) {
>
>     printk\(KERN\_WARNING"Hello, pikachu kernel!\n"\);
>
>     return 0;
>
> }
>
> static void hello\_exit\(void\) {
>
>     printk\(KERN\_INFO"Goodbye, pikachu kernel!\n"\);
>
> }
>
> module\_init\(hello\_init\);
>
> module\_exit\(hello\_exit\);
>
> MODULE\_LICENSE\("GPL"\);
>
> Makefile：
>
> ifneq \($\(KERNELRELEASE\),\)
>
> obj-m := hello.o
>
> else
>
> KDIR := /lib/modules/\`uname -r\`/build
>
> all :
>
>         make -C $\(KDIR\) M=$\(PWD\) modules
>
> clean:
>
>         make -C $\(KDIR\) M=$\(PWD\) clean
>
> endif



make过程遇到的问题：



> make -C /lib/modules/\`uname -r\`/build M=/root/code\_kernel/hello modules
>
> make: \*\*\* /lib/modules/3.10.0-327.el7.x86\_64/build: No such file or directory.  Stop.
>
> make: \*\*\* \[all\] Error 2



  


解决方法：

/lib/modules/3.10.0-327.el7.x86\_64/build没有指向正确的kernel source。建立软连接即可。



> \[root@localhost 3.10.0-327.el7.x86\_64\]\# pwd
>
> /lib/modules/3.10.0-327.el7.x86\_64
>
> \[root@localhost 3.10.0-327.el7.x86\_64\]\# rm build
>
> \[root@localhost 3.10.0-327.el7.x86\_64\]\# ln -sv /usr/src/kernels/3.10.0-514.2.2.el7.x86\_64/ build



  


**注：kernel打印的日志文件在/var/log/messages下。**

## 



2

单独编译KVM模块



##  

### **2.1 由SPEC文件编译出kernel source**

  


从http://vault.centos.org/7.2.1511/os/Source/SPackages/上面下载

kernel-3.10.0-327.el7.src.rpm

截图如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3zoYydJjr11hC52wFhTyje6Y7ddtbXXAicOMyKC7uiahhDF5W063qXBqGXweu1ljroTptPEKEKghH57Wd7jT8Isg/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

解决依赖：



> yum-builddep kernel-3.10.0-327.el7.src.rpm
>
> rpm -ivh kernel-3.10.0-327.el7.src.rpm
>
> yum-builddep /root/rpmbuild/SPECS/kernel.spec



生成kernel source：



> rpmbuild -bp /root/rpmbuild/SPECS/kernel.spec



生成kernel source的路径：



> /root/rpmbuild/BUILD/kernel-3.10.0-327.el7



### **2.2 单独编译KVM模块**

  


进入该kernel source目录：



> /root/rpmbuild/BUILD/kernel-3.10.0-327.el7/linux-3.10.0-327.el7.x86\_64



单独编译：



> make -j8 -C \`pwd\` M=\`pwd\`/arch/x86/kvm modules



  


生成的kvm.ko和kvm-intel.ko在kernel source//arch/x86/kvm目录下。

查看kvm模块：



> \[root@pizhi-kernel ~\]\# lsmod \| grep kvm
>
> kvm\_intel             162153  0
>
> kvm                   525259  1 kvm\_intel



卸载kvm模块：



> modprobe -r kvm\_intel



安装刚刚单独编译的kvm模块：



> cd /root/rpmbuild/BUILD/kernel-3.10.0-327.el7/linux-3.10.0-327.el7.x86\_64/arch/x86/kvm
>
>   
>
>
> insmod kvm.ko
>
> insmod kvm-intel.ko



