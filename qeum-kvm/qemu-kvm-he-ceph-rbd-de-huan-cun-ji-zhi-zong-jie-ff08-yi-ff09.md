先以客户机（Guest OS） 中的应用写本地磁盘为例进行介绍。客户机的本地磁盘，其实是 KVM 主机上的一个镜像文件虚拟出来的，因此，客户机中的应用写其本地磁盘，其实就是写到KVM主机的本地文件内，这些文件是保存在 KVM 主机本地磁盘上。

  先来看看 I/O 协议栈的层次和各层次上的缓存情况。

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT8jzumX6BKAicl9fn7Hud2PYe5RfQ6FhMypr8qglF6Gfib1GcsXQywdsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1   Linux 内核中的缓存

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT2d2Se9ppzOzxoHR0bjIKDLR3icaOFiaEN6TlJDTaHkibcwEfoETTVSSKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

熟悉 Linux Kernel 的人都知道在内核的存储体系中主要有两种缓存，一是 Page Cache，二是 Buffer Cache。Page Cache 是在 Linux IO 栈中为文件系统服务的缓存，而 Buffer Cache 是处于更下层的 Block Device 层，由于应用大部分使用的存储数据都是基于文件系统，因此 Buffer Cache 实际上只是引用了 Page Cache 的数据，而只有在直接使用块设备跳过文件系统时，Buffer Cache 才真正掌握缓存。关于 Page Cache 和 Buffer Cache 更多的讨论参加 What is the major difference between the buffer cache and the page cache?

  


  这些 Cache 都由内核中专门的数据回写线程负责来刷新到块设备中，应用可以使用如 fsync\(2\), fdatasync\(2\) 之类的系统调用来完成强制执行对某个文件数据的回写。像数据一致性要求高的应用如 MySQL 这类数据库服务通常有自己的日志用来保证事务的原子性，日志的数据被要求每次事务完成前通过 fsync\(2\) 这类系统调用强制写到块设备上，否则可能在系统崩溃后造成数据的不一致。而 fsync\(2\) 的实现取决于文件系统，文件系统会将要求数据从缓存中强制写到持久设备中。

  


 仔细地看一下 fsync 函数（来源）： 

fsync\(\) transfers \("flushes"\) all modified in-core data of \(i.e., modified buffer cache pages for\) the file referred to by the file

  


      descriptor fd to the disk device \(or other permanent storage device\) so that all changed information can be retrieved even after the  system crashed or was rebooted.  This includes writing through or flushing a disk cache if present.  The call blocks until the device

  


      reports that the transfer has completed.  It also flushes metadata information associated with the file \(see stat\(2\)\).

  


  


它会 flush 系统中所有的cache，包括 Page cache 和 Disk write cache 以及 RBDCache，将数据放入持久存储。也就说说，操作系统在 flush Page cache 的时候， RBDCache 也会被 flush。 

  


![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT8jzumX6BKAicl9fn7Hud2PYe5RfQ6FhMypr8qglF6Gfib1GcsXQywdsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2   I/O 协议栈层次及缓存

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT2d2Se9ppzOzxoHR0bjIKDLR3icaOFiaEN6TlJDTaHkibcwEfoETTVSSKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2.1  组成

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTZX0xkME0QNcnfPia381bOrhlUrJdTA67ia322cleSMs6TrzahHJI30qg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

主要组成部分：

Guest OS 和 HOST OS Page Cache

Page Cache 是客户机和主机操作系统维护的用来提高存储 I/O 性能的缓存，它是 Linux 虚拟文件系统缓存的一部分，位于操作系统内存中，它是易失性的，因此，在操作系统奔溃或者系统掉电时，这些数据会消失。数据是否写入 Page cache 可以被控制。当会写入 page cache 时，当数据被写入 page cache 后，应用就认为写入完成了，随后的读操作也会从 page cache 中读取数据，这样性能会提高。可以使用 fsync 来将数据从 page cache 中拷贝到持久存储。

  


在 KVM 环境中，host os 和 guest os 都有 page cache，因此，最好是能绕过一个来提高性能。

  


如果 guest os 中的应用使用 direct I/O 方式，guest os 中 page cache 会被绕过。

  


如果 guest os 使用 no cache 方式，host os 的 page cache 会被绕过。

  


该缓存的特点是读的时候，操作系统先检查页缓存里面是否有需要的数据，如果没有就从设备读取，返回给用户的同时，加到缓存一份;写的时候，直接写到缓存去，再由后台的进程定期涮到磁盘去。这样的机制看起来非常的好，在实践中也效果很好。

  


考虑到其易失性，需要考虑它的大小，特别是在 KVM 主机上。现在 KVM 主机的内存可以很大。其内存越大， 那么在 Page cache 中还没有 flush 到磁盘（虚拟或者物理的）的脏数据就越多，其丢失的后果就越严重。默认的话，Linux 2.6.32 在脏数据达到内存的 10% 的时候会自动开始 flush。

Guest Disk （virtual disk device）：客户机虚机磁盘设备

  


QEMU image：QEMU 镜像文件

  


Physical disk cache

  


这是磁盘的 write cache，它会提高数据到存储的写性能。写到 disk write cache 后，写操作会被认为完成了，即使数据还没真正被写入物理磁盘。这样，如果 disk write cache 没有备份电池的话，断电将导致尚未写入物理磁盘的数据丢失。要强制数据被写入磁盘，应用可以通过操作系统可以发出 fsync 命令。因此，disk write cache 会提到写I/O 性能，但是，需要确保应用和存储栈会将数据写入磁盘中。如果 disk write cache 被关闭，那么写性能将下降，但是断电时数据丢失将会避免。

  


Physical disk platter：物理磁盘

  


2.2  GUEST 应用读 I/O 过程

GUEST OS 中的一个应用发出 read request。

OS 在 guest page cache 中检查。如果有（hit），则直接将 data 从 guest page cache 拷贝到 application space。

  


如果没有（miss），请求被转到 guest virtual disk。该 request 会被 QEMU 转化为对 host 上镜像文件的 read request。

  


Host OS 在 HOST Page cache 中检查。如果 hit，则通过 QEMU 将 data 从 host page cache 传到 guest page cache，再拷贝到 application space。

如果没有（miss），则启动 disk （或者 network）I/O 请求去从实际文件系统中读取数据，读到后再写入 host page cache，在写入 guest page cache，再到 GUEST OS application space。

  


从该过程可以看出：

两重 page cache 会对数据重复保存，这会带来内存浪费

  


两重 page cache 也会提高 hit ratio，因为往往 guest page cache 比 host page cache 会小很多

  


QEMU-KVM Linux 支持关闭和开启任一一个 Page cache，也就是说有四种组合模式，分别会带来不同的效果。在各种I/O的过程中，最好是绕过一个或者两个 Page cache。

  


2.3  Guest 应用 写 I/O 过程

写 I/O 过程比较复杂，本文其余部分会详细阐述。从 1.3 表格总结，基本上

  


writeback/unsafe：app ---qemu write---

&gt;

 host page cache  --- os flush --

&gt;

 disk cache --- hw flush ---

&gt;

 disk

none:     app  --- qemu write---

&gt;

 disk write cache  --- hw flush --

&gt;

 disk

writethrough:    app --- qemu write----

&gt;

 host page cache, disk

directsync:  app --- qemu write ---

&gt;

 disk

  


关于 guest os page cache，看起来它主要是作为读缓存，而对于写，没有一种模式是以写入它作为写入结束标志的。

  


![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT8jzumX6BKAicl9fn7Hud2PYe5RfQ6FhMypr8qglF6Gfib1GcsXQywdsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3   客户机磁盘（drive）的缓存模式

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT2d2Se9ppzOzxoHR0bjIKDLR3icaOFiaEN6TlJDTaHkibcwEfoETTVSSKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在 libvirt xml 中使用 'cache' 参数来指定driver的缓存模式，比如：

  


&lt;

disk type='file' device='disk'

&gt;

  \#对于 type，'file' 表示是 host 上的文件，'network' 表示通过网络访问，比如Ceph

  


&lt;

driver name='qemu' type='raw' cache='writeback'/

&gt;

  


  


QEMU/KVM 支持如下这些缓存模式作为 ‘cache’ 的可选值：

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT8D5BCpFeBQcozyYjGVPyB3iamRL1rbjgadIyXcaHJyCt8s39eTdVGUw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

性能：  writeback = unsafe 

&gt;

 none 

&gt;

 writethrough = directsync

  


安全性: writethrough = directsync 

&gt;

 none 

&gt;

 writeback 

&gt;

 unsafe

  


看看性能比较：

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTY9iaVP1XOs42eCotqqDdOsQyzpYotpo0JI1bGX7aF736SbzBbYdhdnQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

基本结论：

各种模式的性能差别非常大

对于数据库这样的应用，使用 directsync 模式，数据直接写入物理磁盘才算成功

对于重要的数据或者小 I/O 的场景，使用 writethrough

对于一般的应用，或者大 I/O 场景，使用 none。这个可以说是大部分情况下的最优选项。

对于丢失了也无所谓的数据，可以使用 writeback

  


![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

4   KVM Write barrier

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

4.1  什么是 KVM write barrier

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

上面的基本结论中，writethrough 是最安全的，但是效率也是最低的。它将数据放在 HOST Page Cache 中，一方面来支持读缓存，另一方面，在每一个 write 操作后，都执行 fsync，确保数据被写入物理存储。只有在数据被写入磁盘后，写操作才会标记为成功。这种模式下，客户机的 virtual storage adapter 会被通知不会使用 writeback 模式，因此，它不会主动发送 fsync 命令，因为它是重复的，不需要的。

  


  那还有没有什么办法使它在保持数据可靠性的同时，使它的效率提高一些呢？答案是 KVM Write barrier 功能。新的 KVM 版本中，启用了 “barrier-passing” 功能，它能保证在不管是用什么缓存模式下，将客户机上应用写入的数据 100% 写入持久存储。

  


  好吧，这真是个神器。。那它是如何实现的呢？以 fio 工具为例，在支持 write barrier 的客户机操作系统上，在使用 direct 和 sync 参数的情况下，会使用这种模式。它在写入部分数据以后，会使得操作系统发出一个 fdatasync 命令，这样 QEMU-KVM 就会将缓存中的数据 flush 到物理磁盘上。

基本过程：

在一个会话中写入数据

发出 barrier request

会话中的所有数据被 flush 到物理磁盘

继续下一个会话

 看起来和 writethrough 差不多是吧。但是它的效率比 writethrough 高。两者的区别在于，writethrough 是每次 write 都会发 fsync，而 barrier-passing 是在若干个写操作或者一个会话之后发 fdatasync 命令，因此其效率更高。

  也可以看到，使用它是有条件的：

KVM 版本较新

客户机操作系统支持：在较新的 Linux 发行版中都会支持

客户机中的文件系统支持 barrier （ext4 支持并默认开启；ext3 支持但默认不开启），而且整个 I/O 协议栈中的各个层次都支持 flush 操作

应用需要在需要的时候发出 flush 指令。

  


  也可以看到，应用在需要的时候发出 flush 指令是关键。一方面，Cache 都由内核中专门的数据回写线程负责来刷新到块设备中；另一方面，应用可以使用如 fsync\(2\), fdatasync\(2\) 之类的系统调用来完成强制执行对某个文件数据的回写。像数据一致性要求高的应用如 MySQL 这类数据库服务通常有自己的日志用来保证事务的原子性，日志的数据被要求每次事务完成前通过 fsync\(2\) 这类系统调用强制写到块设备上，否则可能在系统崩溃后造成数据的不一致。而 fsync\(2\) 的实现取决于文件系统，文件系统会将要求数据从缓存中强制写到持久设备中。类似地，支持 librbd 的QEMU 在适当的时候也会发出 flush 指令。

  


  以 fio 为例，设置有两个参数时，会有 flush 指令发出：

fsync=intHow many I/Os to perform before issuing an fsync\(2\) of dirty data. If 0, don't sync. Default: 0.fdatasync=intLike fsync, but uses fdatasync\(2\) instead to only sync the data parts of the file. Default: 0.

  


  


需要注意的是，频繁的发送（int 值设置的比较小），会影响 IOPS 的值。为了测得最大的IOPS，可以在测试准备阶段发一个sync，然后再收集阶段就不发sync，完全由 RBDCache 自己的机制去 flush；或者需要的话，把 int 值设得比较大，来模拟一些应用场景。

  




4.2  KVM write barrier 和 KVM 缓存模式的结合



考虑到 KVM write barrier 的原理和 KVM 各种缓存模式的原理，显而易见，writeback + barrier 的方式下，可以实现 效率最高+数据安全 这种最优效果。

  


![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTTkVEysZNOYh25LWJXlPAuU4hoHibu1MPicUtOsxUbyHK1gSUzqYTmagQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTibRhYaESNcALIiaNOib3IGZzMZ7d8GZSl3VRlb57dcuSzPhDAzibcnlnkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

5   小结

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTokia4IDour24PU8FVvVQ78Ribsx5Gibtr4BdbFtbzibL2BKx2Kc8lQFhJg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在客户机可以启用 write barrier 时，使用 write-back or nocache + barrier，然后应用会在合适的时候发出 flush 指令。

  


在客户机不支持 write barrier 时，如果对读敏感应用，使用 write-back （可以使用 pagecache）；对需要同步数据的应用，使用 noncache；最安全的情况下，使用 writethrough。

  


对于一些能过备用电池或者别的技术（比如设备上有电容等）保证了在掉电情况下数据也不会丢失的情况下，barrier 最好被禁止。比如企业存储的Adatper，或者 SSD。

  


”If the device does not need cache flushes it should not report requiring flushes, in which case nobarrier will be a noop.“

”With a RAID controller with battery backed controller cache and cache in write back mode, you should turn off barriers - they are unnecessary in this case, and if the controller honors the cache flushes, it will be harmful to performance. But then you \*must\* disable the individual hard disk write cache in order to ensure to keep the filesystem intact after a power failure.“。

  


一个例子是，Ceph OSD 节点上的 SSD 分区，一般都使用 ”nobarrier“参数 来禁用 barrier。 

  


![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTibRhYaESNcALIiaNOib3IGZzMZ7d8GZSl3VRlb57dcuSzPhDAzibcnlnkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

6   Linux page cache 补充

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTokia4IDour24PU8FVvVQ78Ribsx5Gibtr4BdbFtbzibL2BKx2Kc8lQFhJg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Linux 系统中被用于 Page cache 的主内存可以通过 free -m 命令来查看：

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT6iaHHMvxOWLU9WEiaPOurtQX5sx9Kb7XQG7eGHaW8m6880lodZSco46A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

写 Page cache：

（1）当数据被写时，通常情况下，它首先会被写入 page cache，被当作一个  dirty page 来管理。’dirty‘ 的意思是，数据还保存在 page cache 中，还需要被写入底下的持久存储。dirty pages 中的内容会被系统周期性地、以及使用诸如 sync 或者 fsync 的系统调用来写入持久存储。

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTOIAbqr5vt7fpllBia96CJ1aE5uTicFX0lwkDgVowNu3ia3c4CtWXE8ScA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

sync writes any data buffered in memory out to disk. This can include \(but is not limited to\) modified superblocks, modified inodes, and delayed reads and writes.

  


  


（2）当数据被从持久性存储读出时，它也会被写入 page cache。因此，当连续两次读时，第二次会比第一次快，因为第一次读后把数据写入了 page cache，第二次就直接从这里读了。完整过程如下：

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTiblqmHcxPg6XsF7zKqicSXmSmeviavA8zVZ3y4ZVCNAr2J3gtic8INqTAQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

需要注意的是，需要结合应用的需要，来决定是否在写数据时一并写入 page cache。

