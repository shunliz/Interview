1

下载qemu2.7



目前CentOS Linux release 7.2.1511 官方的qemu版本是qemu2.3。很多公司为了提高存储虚拟机等性能都在测试qemu2.7。qemu社区下载地址是http://wiki.qemu.org/Download ，下载后解压至自己的目录。这里路径为

/root/qemu-2.7.0

2

编译qemu2.7



公司服务器目前用的CentOS Linux release 7.2.1511，因为需要涉及到修改qemu源码，所以修改qemu源码，git管理，rpmbuild成rpm包这些都不在一一赘述。

直接上干货：



2.1

解决编译依赖

yum install yum-utils -y   ==&gt;安装yum-builddep工具

这里取巧的方式就是通过下载：

http://vault.centos.org/7.2.1511/virt/Source/kvm-common/ 

下面的qemu-kvm-ev-2.3.0-31.el7\_2.21.1.src.rpm，

yum-builddep qemu-kvm-ev-2.3.0-31.el7\_2.21.1.src.rpm

上面这一步会直接根据srpm包里面的SPEC文件安装依赖。到这里绝大部分的qemu依赖就解决了



2.2

configure

通过srpm包同样提取如下build\_qemu.sh，如下：

\[root@pikachu qemu-2.7.0\]\# cat build\_qemu.sh

\#!/bin/sh

./configure \

    --prefix=/usr \

    --datadir=/usr/share/ \

    --with-confsuffix=/qemu-kvm \

    --libdir=/usr/lib64 \

    --sysconfdir=/etc \

    --interp-prefix=/usr/qemu-%M \

    --localstatedir=/var \

    --libexecdir=/usr/lib64 \

    --extra-ldflags="-Wl,--build-id -pie -Wl,-z,relro -Wl,-z,now" \

    --extra-cflags="-O2 -g -pipe -Wall  -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIE -DPIE" \

    --with-coroutine=ucontext \

    --with-system-pixman \

    --disable-archipelago \

    --disable-bluez \

    --disable-brlapi \

    --disable-cap-ng \

    --enable-coroutine-pool \

    --enable-curl \

    --disable-curses \

    --disable-debug-tcg \

    --enable-docs \

    --disable-gtk \

    --enable-kvm \

    --enable-libiscsi \

    --disable-libnfs \

    --enable-libssh2 \

    --enable-libusb \

    --disable-bzip2 \

    --enable-linux-aio \

    --enable-lzo \

    --disable-opengl \

    --enable-pie \

    --disable-qom-cast-debug \

    --disable-sdl \

    --enable-snappy \

    --disable-sparse \

    --disable-strip \

    --disable-tpm \

    --enable-trace-backend=dtrace \

    --enable-uuid \

    --disable-vde \

    --enable-vhdx \

    --disable-vhost-scsi \

    --disable-virtfs \

    --disable-vnc-jpeg \

    --enable-vnc-png \

    --enable-vnc-sasl \

    --disable-vte \

    --enable-werror \

    --disable-xen \

    --disable-xfsctl \

    --disable-fdt \

    --enable-glusterfs \

    --disable-guest-agent \

    --enable-numa \

    --enable-rbd \

    --enable-rdma \

    --enable-seccomp \

    --enable-spice \

    --enable-usb-redir \

    --enable-tcmalloc \

    --audio-drv-list=pa,alsa \

    --block-drv-rw-whitelist=qcow2,raw,file,host\_device,nbd,iscsi,gluster,rbd,blkdebug \

    --block-drv-ro-whitelist=vmdk,vhdx,vpc,https,ssh \

    --target-list=x86\_64-softmmu \

这里直接将build\_qemu.sh复制至/root/ qemu-2.7.0\(你解压的qemu2.7.0目录\)

执行如下操作：

\[root@pikachu qemu-2.7.0\]\# chmod +x build\_qemu.sh

\[root@pikachu qemu-2.7.0\]\# ./build\_qemu.sh



2.3

make 

\[root@pikachu qemu-2.7.0\]\# make -j32     ==&gt;为了提高编译速度根据当前cpu核数来决定-j后面的number

GEN   x86\_64-softmmu/qemu-system-x86\_64-simpletrace.stp

CC    x86\_64-softmmu/trace/generated-helpers.o

LINK  x86\_64-softmmu/qemu-system-x86\_64

qemu-system-x86\_64就是qemu-2.7.0对应的二进制可行文件

覆盖原来的qemu-kvm，温馨提示：覆盖之前记得备份并且关闭虚拟机！！！

\[root@pikachu qemu-2.7.0\]\# cp x86\_64-softmmu/qemu-system-x86\_64 /usr/libexec/qemu-kvm

注意：

因为Centos官方提供的qemu rpm包安装后的加载路径为/usr/share/qemu-kvm，如果你直接在qemu-2.7.0目录下执行./configure，默认的路径为/usr/local/share/qemu，所以当你virsh start vm-name的时候会提示bios-256k.bin找不到或者其他报错，

解决方法：

用上面的build\_qemu.sh去configure，更加灵活，需要disable enable哪一项非常方便管理。

将/usr/local/share/qemu建立软链接到/usr/share/qemu-kvm

如果按照的是Centos官方提供的qemu rpm包，替换后还会遇到如下问题：提示说不支持改机器的type

qemu-kvm: -machine   pc-i440fx-rhel7.2.0,accel=kvm,usb=off: unsupported machine type

解决方法：

因为rhel和社区所支持不一致，通过virsh edit vm-name，将machine='pc-i440fx-rhel7.2.0' 改成machine='pc'，让其自己去识别即可。

