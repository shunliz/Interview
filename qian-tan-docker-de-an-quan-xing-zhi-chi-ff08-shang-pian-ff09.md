背景

Docker作为最重视安全的容器技术之一，在很多方面都提供了强安全性的默认配置，其中包括：容器root用户的 Capability 能力限制、Seccomp系统调用过滤、Apparmor的 MAC 访问控制、ulimit限制、pid-limits的支持，镜像签名机制等。这篇文章我们就带大家详细了解一下。

![](https://mmbiz.qpic.cn/mmbiz_jpg/3MRbgjUiaA2XEcuS6libMs9hf7qxjc2VPazbkWSFSQgPFbuprkyahaqqckREBVjSsGkJSCnNtnu4fNibSpd6CwDzg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Docker利用Namespace实现了6项隔离，看似完整，实际上依旧没有完全隔离Linux资源，比如/proc 、/sys 、/dev/sd\*等目录未完全隔离，SELinux、time、syslog等所有现有Namespace之外的信息都未隔离。 其实Docker在安全性上也做了很多工作，大致包括下面几个方面：  


**1、Linux内核 Capability 能力限制**

Docker支持为容器设置Capabilities，指定开放给容器的权限。这样在容器中的root用户比实际的root少很多权限。Docker 在0.6版本以后支持将容器开启超级权限，使容器具有宿主机的root权限。

**2、镜像签名机制**

Docker 1.8版本以后提供了镜像签名机制来验证镜像的来源和完整性，这个功能需要手动开启，这样镜像制作者可以在push镜像前对镜像进行签名，在镜像pull的时候，Docker不会pull验证失败或者没有签名的镜像标签。

**3、Apparmor的MAC访问控制**

Apparmor可以将进程的权限与进程capabilities能力联系在一起，实现对进程的强制性访问控制（MAC）。在Docker中，我们可以使用Apparmor来限制用户只能执行某些特定命令、限制容器网络、文件读写权限等功能。

**4、Seccomp系统调用过滤**

使用Seccomp可以限制进程能够调用的系统调用（system call）的范围，Docker提供的默认 Seccomp 配置文件已经禁用了大约 44 个超过 300+ 的系统调用，满足大多数容器的系统调用诉求。

**5、User Namespace隔离**

Namespace为运行中进程提供了隔离，限制他们对系统资源的访问，而进程没有意识到这些限制，为防止容器内的特权升级攻击的最佳方法是将容器的应用程序配置为作为非特权用户运行，对于其进程必须作为容器中的 root 用户运行的容器，可以将此用户重新映射到 Docker 主机上权限较低的用户。映射的用户被分配了一系列 UID，这些 UID 在命名空间内作为从 0 到 65536 的普通 UID 运行，但在主机上没有特权。

**6、SELinux**

SELinux主要提供了强制访问控制（MAC），即不再是仅依据进程的所有者与文件资源的rwx权限来决定有无访问能力。能在攻击者实施了容器突破攻击后增加一层壁垒。Docker提供了对SELinux的支持。

**7、pid-limits的支持**

在说pid-limits前，需要说一下什么是fork炸弹\(fork bomb\)，fork炸弹就是以极快的速度创建大量进程，并以此消耗系统分配予进程的可用空间使进程表饱和，从而使系统无法运行新程序。说起进程数限制，大家可能都知道ulimit的nproc这个配置，nproc是存在坑的，与其他ulimit选项不同的是，nproc是一个以用户为管理单位的设置选项，即他调节的是属于一个用户UID的最大进程数之和。这部分内容下一篇会介绍。Docker从1.10以后，支持为容器指定--pids-limit 限制容器内进程数，使用其可以限制容器内进程数。

**8、其他内核安全特性工具支持**

在容器生态的周围，还有很多工具可以为容器安全性提供支持，比如可以使用 Docker bench audit tool（工具地址：https://github.com/docker/docker-bench-security）检查你的Docker运行环境，使用Sysdig Falco（工具地址：https://sysdig.com/opensource/falco/）来检测容器内是否有异常活动，可以使用GRSEC 和 PAX来加固系统内核等等。

这次分享我们带大家了解一下Docker对前四项是如何安全支持的，下一篇文章会带大家了解剩余内容。

1**Linux内核Capability能力限制**

Capabilities简单来说，就是指开放给进程的权限，比如允许进程可以访问网络、读取文件等。Docker容器本质上就是一个进程，默认情况下，Docker 会删除 必须的 capabilities 外的所有 capabilities，可以在 Linux 手册页 中看到完整的可用 capabilities 列表。Docker 0.6版本以后支持在启动参数中增加 --privileged 选项为容器开启超级权限。

Docker支持Capabilities对于容器安全意义重大，因为在容器中我们经常会以root用户来运行，使用Capability限制后，容器中的 root 比真正的 root用户权限少得多。这就意味着，即使入侵者设法在容器内获取了 root 权限，也难以做到严重破坏或获得主机 root 权限。

当我们在docker run时指定了--privileded 选项，docker其实会完成两件事情：

* 获取系统root用户所有能力赋值给容器；

* 扫描宿主机所有设备文件挂载到容器内。

下面我们给大家实际演示一下：

当执行docker run 时未指定--privileded 选项

```
lynzabo@ubuntu:~$ docker run --rm --name def-cap-con1 -d alpine /bin/sh -c "while true;do echo hello; sleep 1;done"
f216f9261bb9c3c1f226c341788b97c786fa26657e18d7e52bee3c7f2eef755c
lynzabo@ubuntu:~$ docker inspect def-cap-con1 -f '{{.State.Pid}}'
43482
lynzabo@ubuntu:~$ cat /proc/43482/status | grep Cap
CapInh:    00000000a80425fb
CapPrm:    00000000a80425fb
CapEff:    00000000a80425fb
CapBnd:    00000000a80425fb
CapAmb:    0000000000000000
lynzabo@ubuntu:~$
lynzabo@ubuntu:~$ docker exec def-cap-con1 ls /dev
core  fd  full  mqueue  null  ptmx  pts  random  shm  stderr  stdin  stdout  tty  urandom  zero  ...总共15条
lynzabo@ubuntu:~$
```

如果指定了--privileded 选项

```
lynzabo@ubuntu:~$ docker run --privileged --rm --name pri-cap-con1 -d alpine /bin/sh -c "while true;do echo hello; sleep 1;done"
ad6bcff477fd455e73b725afe914b82c8aa6040f36326106a9a3539ad0be03d2
lynzabo@ubuntu:~$ docker inspect pri-cap-con1 -f '{{.State.Pid}}'
44312
lynzabo@ubuntu:~$ cat /proc/44312/status | grep Cap
CapInh:    0000003fffffffff
CapPrm:    0000003fffffffff
CapEff:    0000003fffffffff
CapBnd:    0000003fffffffff
CapAmb:    0000000000000000
lynzabo@ubuntu:~$ docker exec pri-cap-con1 ls /dev
agpgart  autofs  bsg  btrfs-control  bus  core  cpu_dma_latency  cuse  dmmidi  dri  ecryptfs
...总共186条
lynzabo@ubuntu:~$
```

对比/proc/$pid/status ，可以看到两个容器进程之间能力位图的差别，加上--privileged 的能力位图与超级用户的能力位图一样。再对比增加--privileged后目录/dev 下文件变化，可以看到，加了特权后，宿主机所有设备文件都挂载在容器内。

我们可以看到，使用--privileged 参数授权给容器的权限太多，所以需要谨慎使用。如果需要挂载某个特定的设备，可以通过--device方式，只挂载需要使用的设备到容器中，而不是把宿主机的全部设备挂载到容器上。例如，为容器内挂载宿主机声卡：

```
$ docker run --device=/dev/snd:/dev/snd …
```

此外，可以通过--add-cap 和 --drop-cap 这两个参数来对容器的能力进行调整，以最大限度地保证容器使用的安全。

例如，给容器增加一个修改系统时间的命令：

```
$ docker run --cap-drop ALL --cap-add SYS_TIME ntpd /bin/sh
```

查看容器PID，执行getpcaps PID查看进程的能力，执行结果如下：

    [root@VM_0_6_centos ~]# getpcaps 652
    Capabilities for `652': = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_time,...
    [root@VM_0_6_centos ~]#

可以看到容器中已经增加了sys\_time 能力，可以修改系统时间了。

2**Docker镜像签名机制**

当我们执行docker pull 镜像的时候，镜像仓库再验证完用户身份后，会先返回一个manifest.json文件，其中包含了镜像名称、tag、所有layer层SHA256值，还有镜像的签名信息，然后docker daemon会并行的下载这些layer层文件。Docker 1.8以后，提供了一个数字签名机制——content trust来验证官方仓库镜像的来源和完整性,简单来说就是镜像制作者制作镜像时可以选择对镜像标签（tag）进行签名或者不签名，当pull镜像时，就可以通过这个签名进行校验，如果一致则认为数据源可靠，并下载镜像。

默认情况下，这个content trust是被关闭了的，你需要设置一个环境变量来开启这个机制，即：

```
$ export DOCKER_CONTENT_TRUST=11
```

当content trust机制被开启后，docker不会pull验证失败或者没有签名的镜像标签。当然也可以通过在pull时加上--disable-content-trust来暂时取消这个限制。

3**Apparmor的MAC访问控制**

AppArmor和SELinux都是Linux安全模块，可以将进程的权限与进程capabilities能力联系在了一起，实现对进程的强制性访问控制（MAC）。由于SELinux有点复杂，经常都被人直接关闭，而AppArmor就相对要简单点。Docker官方也推荐这种方式。

Docker 自动为容器生成并加载名为 docker-default 的默认配置文件。在 Docker 1.13.0和更高版本中，Docker 二进制文件在 tmpfs 中生成该配置文件，然后将其加载到内核中。在早于 1.13.0 的 Docker 版本上，此配置文件将在 /etc/apparmor.d/docker 中生成。docker-default 配置文件是运行容器的默认配置文件。它具有适度的保护性，同时提供广泛的应用兼容性。

> 注意：这个配置文件用于容器而不是 Docker 守护进程。运行容器时会使用 docker-default 策略，除非通过 security-opt 选项覆盖。

下面我们使用Nginx做演示，提供一个自定义AppArmor 配置文件：

1、创建自定义配置文件，假设文件路径为 /etc/apparmor.d/containers/docker-nginx 。

```
#include 
<
tunables/global
>


profile docker-nginx flags=(attach_disconnected,mediate_deleted) {
  #include 
<
abstractions/base
>

  ...
  deny network raw,
  ...
  deny /bin/** wl,
  deny /root/** wl,
  deny /bin/sh mrwklx,
  deny /bin/dash mrwklx,
  deny /usr/bin/top mrwklx,
  ...
}
```

2、加载配置文件

```
$ sudo apparmor_parser -r -W /etc/apparmor.d/containers/docker-nginx
```

3、使用这个配置文件运行容器

```
$ docker run --security-opt "apparmor=docker-nginx" -p 80:80 -d --name apparmor-nginx nginx12
```

4、进入运行中的容器中，尝试一些操作来测试配置是否生效：

```
$ docker container exec -it apparmor-nginx bash1
root@6da5a2a930b9:~# ping 8.8.8.8
ping: Lacking privilege for raw socket.

root@6da5a2a930b9:/# top
bash: /usr/bin/top: Permission denied

root@6da5a2a930b9:~# touch ~/thing
touch: cannot touch 'thing': Permission denied

root@6da5a2a930b9:/# sh
bash: /bin/sh: Permission denied
```

可以看到，我们通过 apparmor 配置文件可以对容器进行保护。

4**Seccomp系统调用过滤**

Seccomp是Linux kernel 从2.6.23版本开始所支持的一种安全机制，可用于限制进程能够调用的系统调用（system call）的范围。在Linux系统里，大量的系统调用（systemcall）直接暴露给用户态程序，但是，并不是所有的系统调用都被需要，而且不安全的代码滥用系统调用会对系统造成安全威胁。通过Seccomp，我们限制程序使用某些系统调用，这样可以减少系统的暴露面，同时使程序进入一种“安全”的状态。每个进程进行系统调用（system call）时，kernel 都会检查对应的白名单以确认该进程是否有权限使用这个系统调用。从Docker1.10版本开始，Docker安全特性中增加了对Seccomp的支持。  


使用Seccomp的前提是Docker构建时已包含了Seccomp，并且内核中的CONFIG\_SECCOMP已开启。可使用如下方法检查内核是否支持Seccomp：

    $ cat /boot/config-`uname -r` | grep CONFIG_SECCOMP=
    CONFIG_SECCOMP=y

默认的 seccomp 配置文件为使用 seccomp 运行容器提供了一个合理的设置，并禁用了大约 44 个超过 300+ 的系统调用。它具有适度的保护性，同时提供广泛的应用兼容性。默认的 Docker 配置文件可以在moby源码profiles/seccomp/下找到。



默认seccomp profile片段如下：

```
{
 "defaultAction": "SCMP_ACT_ERRNO",
 "archMap": [
  {
   "architecture": "SCMP_ARCH_X86_64",
   "subArchitectures": [
    "SCMP_ARCH_X86",
    "SCMP_ARCH_X32"
   ]
  },=
  ...
 ],
 "syscalls": [
  {
   "names": [
    "reboot"
   ],
   "action": "SCMP_ACT_ALLOW",
   "args": [],
   "comment": "",
   "includes": {
    "caps": [
     "CAP_SYS_BOOT"
    ]
   },
   "excludes": {}
  },
  ...
 ]
}
```

seccomp profile包含3个部分：默认操作，系统调用所支持的Linux架构和系统调用具体规则\(syscalls\)。对于每个调用规则，其中name是系统调用的名称，action是发生系统调用时seccomp的操作，args是系统调用的参数限制条件。比如上面的“SCMP\_ACT\_ALLOW”action代表这个进程这个系统调用被允许，这个call，允许进程可以重启系统。  


> 实际上，该配置文件是白名单，默认情况下阻止访问所有的系统调用，然后将特定的系统调用列入白名单。

seccomp有助于以最小权限运行 Docker 容器。不建议更改默认的 seccomp 配置文件。

运行容器时，如果没有通过 --security-opt 选项覆盖容器，则会使用默认配置。例如，以下显式指定了一个策略：

```
$ docker run --rm \
             -it \
             --security-opt seccomp=/path/to/seccomp/profile.json \
             hello-seccomp
```

Docker 的默认 seccomp 配置文件是一个白名单，它指定了允许的调用。Docker文档列举了所有不在白名单而被有效阻止的重要（但不是全部）系统调用以及每个系统调用被阻止的原因，详细内容链接：

https://docs.docker.com/engine/security/seccomp/\#significant-syscalls-blocked-by-the-default-profile

这篇文章的分享就到这里，下一篇我们会解释Docker安全性的另外四个方面的支持，希望本次分享对大家有所帮助。

