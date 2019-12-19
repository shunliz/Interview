**登录centos官网，分别下载版本源码包： **  


qemu-kvm-1.5.3-60.el7.src.rpm 

libvirt-1.1.1-29.el7.src.rpm 

要安装rpmbuild这个包

  


**编译安装qemu**

  


首先

安装源码：rpm -ivh qemu-kvm-0.12.1.2-2.415.el6.src.rpm，

要编译的源码和需要的spec文件都自动放到了/root/rpmbuild/目录下。

  


然后

修改/root/rpmbuild/SPECS/qemu-kvm.spec这个文件，将两处（--disable-sdl）改为（--enable-sdl）。

  


之后

编译rpm包：rpmbuild --target=x86\_64 -bb /root/rpmbuild/SPECS/qemu-kvm.spec编译好的rpm包在/root/rpmbuild/RPMS/x86\_64/中

  


最后

安装生成的rpm包，如果系统中存在旧版本，则强制安装就可以替代之前的版本（rpm -ivh \*.rpm --force），一般只需要安装和qemu有关的rpm包即可。

  


安装后运行/usr/libexec/qemu-kvm，应该就直接看到qemu的SDL窗口，如果提示缺少依赖包或者少库，则可以依次安装。

  


  


**编译安装libvirt**

  


过程与qemu相同，修改libvirt的spec文件把下面两行删除：

  


_--with-qemu-user=%{qemu\_user}  _

_--with-qemu-group=%{qemu\_group} _

  


就可以进行编译和安装（有些生成包不是必须的，可以不装）。

  


  


**现在就可以用SDL窗口来打开虚拟机了**

  


不过打开之前需要进行一些设置。如果你用了virt-manager工具，则将现在使用的Display硬件删除，然后添加Graphics硬件，选择本地SDL选项，运行虚拟机就可以直接在SDL中看到虚拟机了。

  


如果virt-manager链接不上SDL，打印类似Could not initialize SDL之类的，先setenforce 0一下。

  


如果不使用virt-manager软件，而在shell中使用libvirt来打开sdl，则需要在本地虚拟机的xml文件中添加或者修改graphics标签：

  


_&lt;graphics type='sdl' display=':0' xauth='/root/.Xauthority'/&gt;_

  


其中display和xauth的值可以通过当前终端获得：

  


输入命令：env，其中会有两行：

  


_DISPLAY=:0_

_XAUTHORITY=/run/gdm/auth-for-root-oAEUYz/database_  


  


设置到graphics标签中即可，如果要全屏打开，则再添加选项fullscreen="yes"

  


如果virt-manager链接不上SDL，打印类似Could not initialize SDL之类的，先setenforce 0一下。

