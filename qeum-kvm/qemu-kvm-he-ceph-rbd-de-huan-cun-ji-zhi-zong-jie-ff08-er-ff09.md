QEMU 能够支持将主机的一个块设备映射给客户机，但是从 0.15 版本开始，就不再需要先将一个 Ceph volume 映射到主机再给客户机了。现在，QEMU 可以直接通过 librbd 来象 virtual block device 一样访问 Ceph image。这既提高了性能，也使得可以使用 RBDCache 了。

  


  RBDCache 是 Ceph 的块存储接口实现库 Librbd 用来在客户端侧缓存数据的目的，它主要提供了读数据缓存，写数据汇聚写回的目的，用来提高顺序读写的性能。需要说明的是，Ceph 既支持以内核模块的方式来实现对 Linux 动态增加块设备，也支持以 QEMU Block Driver 的形式给使用 QEMU 虚拟机增加虚拟块设备，而且两者使用不同的库，前者是内核模块的形式，后者是普通的用户态库，本文讨论的 RBDCache 针对后者，前者使用内核的 Page Cache 达到目的。

  


![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT8jzumX6BKAicl9fn7Hud2PYe5RfQ6FhMypr8qglF6Gfib1GcsXQywdsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1  librbd I/O 协议栈

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTcbDal6CtiaAic3YQNo8zPxibibu284bxxkKyB4b9iaRaRxljnLuEyBYIIkQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTl437iaInnF6SxiaVseHB1ArgfjCcn22yAojcZias2btQ556IcwzQNrv1A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 从这个栈可以看出来，RBDCache 类似于磁盘的 write cache。它应该有三个功能：

  


写缓存：开启时，librdb 将数据写入 RBDCache，然后在被 flush 到 Ceph 集群，其效果就是多个写操作被合并，但是有一定的时间延迟。

读缓存：数据会在缓存中被保留一段时间，这期间的 librbd 读数据的话，会直接从缓存中读取，提高读效率。

  


合并写操作：对同一个 OSD 上的多个写操作，应该会合并为一个大的写操作，提高写入效率。 ”Due to several objects map to the same physical disks, the original logical sequential IO streams mix together \(green, orange, blue and read blocks\).  来源“

  


  因此，需要注意的是，理论上，RBDCache 对顺序写的效率提升应该非常有帮助，而对随机写的效率提升应该没那么大，其原因应该是后者合并写操作的效率没前者高（也就是能够合并的写操作的百分比比较少）。具体效果待测试。

  


  在使用 QEMU 实现的 VM 来使用 RBD 块设备，那么 Linux Kernel 中的块设备驱动是 virtio\_blk，它会对块设备各种请求封装成一个消息通过 virtio 框架提供的队列发送到 QEMU 的 IO 线程，QEMU 收到请求后会转给相应的 QEMU Block Driver 来完成请求。当 QEMU Block Driver 是 RBD 时，缓存就会交给 Librbd 自身去维护，也就是一直所说的 RBDCache；用户在使用本地文件或者 Host 提供的 LVM 分区时，跟 RBDCache 同样性质的缓存包括了 Guest Cache 和 Host Page Cache，见本文第一部分的描述。

  


![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT8jzumX6BKAicl9fn7Hud2PYe5RfQ6FhMypr8qglF6Gfib1GcsXQywdsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2  

RBDCache 的原理

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTws8fdajforwtK3yicSEFsclsFbicyADmh0ATaprpFyC5O98icmF784n0Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



2.1 RBDCache 的配置



在 ceph.conf 中，设置 rbd cache = true 即可以启用 RBDCache。它有以下几个主要的配置参数：

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTLrJYojYZrEFvBr6bOTSrDIbvkn4FpZBiak3GssXJEfy0jHfnfE1lhsg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 可见，默认情况下：

在主机操作系统内内存内会分配 32MiB 的空间用于 RBD 做缓存使用

  


允许最大的脏数据大小为 24MiB，超过的话，可能会阻止继续写入（需要确认）

在脏数据总共有 16MiB 时，开始回写过程，将数据写入Ceph集群

  


在单个脏数据（目前在 Librbd 用户态库中主要以 Object Buffer Extent 为基本单位进行缓存，这里的粒度应该是 Object Buffer Extent）存在超过 1 秒时，对它启用回写

  也能看出，RBDCache 从空间和时间来方面，在效率和数据有效性之间做平衡。

  


几个重要的注意事项：

  


（1）QEMU 和 ceph 配置项的相互覆盖问题

http://ceph.com/docs/master/rbd/qemu-rbd/\#qemu-cache-options

  


在没有在 Ceph 配置文件中显式配置 RBD Cache 的参数（尽管Ceph 支持配置项的默认值，但是，看起来，是否在Ceph配置文件中写还是不写，会有不同的效果。。真绕啊。。）时，QEMU 的 cache 配置会覆盖 Ceph 的默认配置。

  


qemu driver 'writeback' 相当于 rbd\_cache = true

qemu driver ‘writethrough’ 相当于 ‘rbd\_cache = true,rbd\_cache\_max\_dirty = 0’

qemu driver ‘none’ 相当于 rbd\_cache = false

一个典型场景是，在 nova.conf 中配置了 ”cache=writeback”，而没有在客户端节点上配置 Ceph 配置文件，这时候将直接打开 RBDCache 并使用 writeback 模式，而不是先 writethrough 后 writeback。

  


在在 Ceph 配置文件中显式配置了缓存模式的时候，Ceph 的 cache 配置会覆盖 QEMU 的 cache 配置。

如果在 QEMU 的命令行中使用了 cache 配置，则它会覆盖 Ceph 配置文件中的配置。

优先级：QEMU 命令行中的配置 

&gt;

 Ceph 文件中的显式配置 

&gt;

 QEMU 配置 

&gt;

 Ceph 默认配置

  


（2）在启用 RBDCache 时，必须在 QEMU 中配置 ”cache=writeback”，否则可能会导致数据丢失。在使用文件系统的情况下，这可能会导致文件系统损坏。

  


Important 

If you set rbd\_cache=true, you must set cache=writeback or risk data loss. Without cache=writeback, QEMU will not send flush requests to librbd. If QEMU exits uncleanly in this configuration, filesystems on top of rbd can be corrupted.

http://ceph.com/docs/master/rbd/qemu-rbd/\#running-qemu-with-rbd

  


（3）使用 raw 格式的 Ceph 卷设备 “ 

&lt;

driver name='qemu' type='raw' cache='writeback'/

&gt;

“

http://ceph.com/docs/master/rbd/qemu-rbd/\#creating-images-with-qemu

理论上，你可以使用其他 QEMU 支持的格式比如 qcow2 或者 vmdk，但是它们会带来 overhead

  


The raw data format is really the only sensible format option to use with RBD. Technically, you could use other QEMU-supported formats \(such as qcow2 or vmdk\), but doing so would add additional overhead, and would also render the volume unsafe for virtual machine live migration when caching \(see below\) is enabled.

  


（4）在新版本的 Ceph 中（将来的版本，尚不知版本号），Ceph 配置项 rbd cache 将会被删除，RBDCache 是否开启将由 QEMU 配置项决定。

也就是说，如果 QEMU 中设置 cache 为 ‘none’ 的话， RBDCache 将不会被使用；设置为 ‘writeback’ 的话，RBDCache 将会被启用。参考链接：ceph : \[client\] rbd cache = true override qemu cache=none\|writeback。

  


（5）对 Nova 来说，不设置 disk\_cachemode 值的话，默认的 driver 的 cache 模式是 ‘none’。但是，在不支持 ‘none’ 模式的存储系统上，会改为使用 ‘writethrough’ 模式。

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTEXsMxzVrS2ja9BDBn1Sjg53xAEE1uJxkplj61wIfuhVthIF0rGn1Pg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



2.2 缓存中的数据被 flush 到 Ceph cluster



有两种类型的 flush：

RBD 主动的，在 RBDCache 规定的空间或者数据保存时间达到阈值之后，会触发回写

  


RBD 被动的，librbd 的 flush 接口被调用，全部缓存中的数据也会被回写。又可以细分为两种类型：

QEMU 在合适的时候会自动发出 flush：QEMU 作为最终使用 Librbd 中 RBDCache 的用户，它在 VM 关闭、QEMU 支持的热迁移操作或者 RBD 块设备卸载时都会调用 QEMU Block Driver 的 Flush 接口，确保数据不会被丢失。因此，此时，需要用户在使用了开启 RBDCache 的 RBD 块设备 VM 时需要给 QEMU 传入 “cache=writeback” 确保 QEMU 知晓有缓存的存在，不然 QEMU 会认为后端并没有缓存而选择将 Flush Request 忽略。

  


应用发出 flush，比如 fio，可以设置 fdatasync 为一个大于零的整数，从而在若干次写操作后执行fdatasync。（ fdatasync=int Like fsync, but uses fdatasync\(2\) instead to only sync the data parts of the file. Default: 0“

  


  关于第二种 flush，这里的一个问题是，什么时候会有这种主动 flush 指定发出。有文章说，”QEMU 作为最终使用 Librbd 中 RBDCache 的用户，它在 VM 关闭、QEMU 支持的热迁移操作或者 RBD 块设备卸载时都会调用 QEMU Block Driver 的 Flush 接口“。同时，一些对数据的安全性敏感的应用也可以通过操作系统在需要的时候发出 flush 指定，比如一些数据库系统。你可以使用 fio 工具的 fdatasync 参数在指定的写入操作后发出 fdatasync 指令。具体效果还待测试。

  librados 的 flush API：

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCT0RWV5NsVpF2ZoLr8ucueawwqqhyqfGaNvYEH08dXHicwib4KLlLRaoJA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

下面是一个 Linux 系统上文件操作的伪代码（来源）。可见，该程序知道只有在 fdatasync 执行成功后，数据才算写入成功。

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTULwdySSYyiasVjtpjjm5cO15jpI7HsPv9EfH5hw3YVlaqGg3F8ospfA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



2.3  RBDCache 中数据的易失性和 librbd rbd\_cache\_writethrough\_until\_flush 配置项



因为 RBDCache 是利用内存来缓存数据，因此数据也是易失性的。那么，最安全的是，设置 rbd\_cache\_max\_dirty = 0，就是不缓存数据，相当于 writethrough 的效果。很明显，这没有实现 RBDCache 的目的。 

  


  另外，Ceph 还提供 rbd\_cache\_writethrough\_until\_flush 选项，它使得 RBDCache 在收到第一个 flush 指令之前，使用 writethrough 模式，透传数据，避免数据丢失；在收到第一个 flush 指令后，开始 writeback 模式，通过 KVM barrier 功能来保证数据的可靠性。   

  


  该选项的含义：

This option enables the cache for reads but does writethrough until we observe a FLUSH command come through, which implies that the guest OS is issuing barriers.  This doesn't guarantee they are doing it properly, of course, but it means they are at least trying.  Once we see a flush, we infer that writeback is safe.

  


  


  该选项的默认值到底是 true 还是 false 比较坑爹：

ceph/librbd 在 0.80 版本中添加该选项，默认值是 false （代码）

ceph/librbd 在 0.87 版本中将默认值修改为 true （代码）

  


  因此，你在使用不同版本的 librbd 情况下使用默认配置时，其 IOPS 性能是有很大的区别的：

0.80 版本中，一直是 writeback，IOPS 会从头就很好；

0.87 版本中，开始是 writethrough，在收到第一个操作系统发来的 flush 后，转为 writeback，因此，IOPS 是先差后好。

  在实现上，

  


（1）收到第一个flush 之前，相当于 rbd\_cache\_max\_dirty 被设置为0 了：

  


uint64\_t init\_max\_dirty = cct-&gt;\_conf-&gt;rbd\_cache\_max\_dirty;

      if \(cct-&gt;\_conf-&gt;rbd\_cache\_writethrough\_until\_flush\)

           init\_max\_dirty = 0;

（2）收到第一个 flush 之后，就转为 writeback 了

if \(object\_cacher &&cct-&gt;\_conf-&gt;rbd\_cache\_writethrough\_until\_flush\) {

 md\_lock.get\_read\(\);      bool flushed\_before = flush\_encountered;

 md\_lock.put\_read\(\);

 uint64\_t max\_dirty = cct-&gt;\_conf-&gt;rbd\_cache\_max\_dirty;

  


![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTibRhYaESNcALIiaNOib3IGZzMZ7d8GZSl3VRlb57dcuSzPhDAzibcnlnkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3 小结

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTws8fdajforwtK3yicSEFsclsFbicyADmh0ATaprpFyC5O98icmF784n0Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

各种配置下的Ceph RBD 缓存效果：

![](https://mmbiz.qpic.cn/mmbiz/THOMb1XdkLOEjZB3EYO1Hnm14EPubCCTqckVTgrygLkBhZLYCm4ibooYDvGB6V6jiaa3GZRjvk0rfCCEXkjcJ18Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

