High Scalability总结的top20性能瓶颈翻译，并按照自己的理解做了相应的解释。本人能力有限，欢迎拍砖。

  
数据库类:

  


Working size exceeds available RAM

工作任务内存大小超过物理内存大小。工作任务内存过大，导致查询分析器没有做够的内存分析SQL，缓存数据。必然导致这个效率低下。



Long & short running querie

过大或者过小的查询任务。过大的任务，导致一个任务长期占有锁对象，导致其它任务不能访问，会严重影响数据库性能。过小的任务，导致任务切换开销过大，不能



充分利用缓存，导致系统性能下降。



Write-write conflicts

写－写冲突。写数据库需要加排他锁，写频繁，排它锁不能并行，严重影响系统性能。和第一条相似，有些数据库的实现，长的查询也可能加排它锁，也会和写操作冲突，导致性能下降，再次说明大任务的弊端。



Large joins taking up memory

大的连接占用内存。两个大表连接，导致中间结果奇大无比，直接导致执行效率低下。比如，两个10000条记录的表做连接，那么中间结果会是10000\*10000.所以两个表连接一定要过滤后再连接。比如有10000个顾客，10000件商品的交易记录。查询购买过top100商品花费最多的100名顾客，如果不过直接连接再求结果，会从10000\*10000条记录中取100条。先过滤顾客和商品后，各个表为100条，连接后是10000，从10000中取100将会大大提升效率。尽量避免连接的表过多，我见过复杂的业务逻辑直接关联了10多张表。太恐怖。



虚拟化:

Sharing a HDD, disk seek death

不要对一个硬盘做sharding。这样会导致读取一个数据，本来磁头寻道后连续读取就可以，sharding后反而不连续，导致磁头不停移动，硬盘数据读取速度明显下降。



Network I/O fluctuations in the cloud

在云环境中网络I/O突发脉动。由于云中虚拟的系统最终网络I/O是通过物理网络出去，某一段时间如果网络流量突发，所有虚拟机都要发送数据，导致物理网卡超负荷，产生严重丢包，上次应用如果还不停的重发，效率将明显下降。虚拟化环境中最好，所有虚拟错开时间使用网络，随时保证网卡达到额定的负荷是最理想情况。



编程:



Threads: deadlocks, heavyweight as compared to events, debugging, non-linear scalability, etc...

线程：死锁，不能线性扩展，调试。这条只理解线程环境需要避免死锁。



Event driven programming: callback complexity, how-to-store-state-in-function-calls, etc...

事件驱动编程：callback函数太过复杂，怎么在callback中存储状态。

  


Lack of profiling, lack of tracing, lack of logging

没有性能数据，没有调试手段，没有日志



One piece can't scale, SPOF, non horizontally scalable, etc...

单个模块不可扩展，存在单点故障，不能水平扩展



Stateful apps

有状态的app。线程中的任务要尽量写成无状态。如果app有状态，就会涉及到加锁，加锁就会影响性能，降低并发度。



Bad design : The developers create an app which runs fine on their computer. The app goes into production, and runs fine, with a couple of users. 

Months/Years later, the application can't run with thousands of users and needs to be totally re-architectured and rewritten.

不好的设计：开发人员开发了一个应用，在自己机器跑的好好的，在生产环境，少量用户也可以很好的运行。但是经过一段时间，系统不能支撑成千上万的用户，需要完全重写



Algorithm complexity

算法复杂。能用简单的算法解决的不要用复杂的算法解决。简单就是美。否则就是后来人和维护人员的噩梦。



Stack space

栈空间。开发人员需要随时关心是否会存在栈溢出的情况。最蠢的情况莫过无限递归，直接导致系统崩溃。



磁盘:

  


Local disk access

相对于内存访问，磁盘访问的代价是昂贵的。



Random disk I/O -&gt;disk seek

磁盘尽可能顺序访问



Disk fragmentation

数据尽量连续存储，保证磁盘顺序访问



SSDs performance drop once  data written is greater than SSD size

一旦SSD写入的数据大于SSD容量，性能下降。



OS:

  


Fsync flushing, linux buffer cache filling up

及时fsync将数据填满缓存。



TCP buffers too small

TCP的缓冲区太小。我同事前两天反映SSL socket在缓冲区太小的情况下会存在大包丢包的现象，将缓冲区改为64k就ok了。



File descriptor limits

文件描述符限制，操作系统都对可以同时打开的文件描述符数量有限制。毕竟打开太多会占用过多的系统空间，导致系统运行不稳定。必要时可以修改可以同时打开的文件描述符数量。



Power budget

电源预算。  


缓存:

Not using memcached \(database pummeling\)没有使用memcached，导致所有前端请求都直接穿透到后端数据库，导致数据库崩溃。

  
In HTTP: headers, etags, not gzipping, etc..

HTTP协议需要充分利用headers，设置合理的header参数。合理利用etags设计缓存策略，通过gzip减小网络传输的文件大小。

  
Not utilising the browser's cache enough

没有充分利用浏览器的缓存，通过etags等手段可以控制浏览器取本地缓存。原理上浏览器可以通过etags等的值，先从本地缓存取页面，如果有更新，再到网站的前端缓存取，再到真正的服务器取。这又是另外一个大的可能。有兴趣可以看看CDN相关的资料。



Byte code caches \(e.g. PHP\)

字节码缓存，比如PHP。java的class文件等。

  
L1/L2 caches. This is a huge bottleneck. Keep important hot/data in L1/L2. This spans so much: snappy for network I/O, column DBs run algorithms 

  
directly on compressed data, etc. Then there are techniques to not destroy your TLB. The most important idea is to have a firm grasp on computer 

  
architecture in terms of CPUs multi-core, L1/L2, shared L3, NUMA RAM, data transfer bandwidth/latency from DRAM to chip, DRAM caches DiskPages, 

  
DirtyPages, TCP packets travel thru CPU&lt;-&gt;DRAM&lt;-&gt;NIC.

L1/L2缓存。（完全不理解作者怎么解决这个瓶颈）这是一个大的瓶颈。关键数据存储在L1/L2中。这涉及到很多：snappy网络I/O,列数据库直接在压缩数据上运行算法

。有办法不销毁你的TLB。最重要的思想是紧紧的抓住计算机的体系结构，涉及多核CPU，L1/L2,shared L3,NUMA RAM,从DRAM到芯片数据传输带宽和延迟，DRAM缓存的磁盘页，脏页，流经CPU

&lt;

-

&gt;

DRAM

&lt;

-

&gt;

NIC的TCP包。



CPU:

  


CPU overload

  


CPU超负荷了



Context switches -

&gt;

 too many threads on a core, bad luck w/ the linux scheduler, too many system calls, etc...

  


上下文切换过于频繁。单核上开启过多线程，linux的调度器更悲摧。太多的系统调用等。



IO waits -

&gt;

 all CPUs wait at the same speed

  


CPU没充分利用，都在等待慢速I/O.



CPU Caches: Caching data is a fine grained process \(In Java think volatile for instance\), in order to find the right balance between having 



multiple instances with different values for data and heavy synchronization to keep the cached data consistent.

  


CPU缓存。缓存数据是一个为了平衡两者之间：不通实例有不同的值和繁重的同步缓存保持数据一致，而精心设计的一个进程。

  


网络:

NIC maxed out, IRQ saturation, soft interrupts taking up 100% CPU

  


网卡爆棚了，中段饱和了，软中断占据了所有CPU时间。



DNS lookups

  


DNS查询，DNS查询是一个费时的工作。



Dropped packets

  


存在丢包。发包过快，收报过慢。接收缓存过小都可能导致丢包。



Unexpected routes with in the network

  


网络中存在非预期的路由。比如引发网络风暴，路由环路等问题。



Network disk access

  


网路硬盘访问。网络接口慢，磁盘接口也慢，两者结合，慢上加慢。



Shared SANs

  


共享SAN。SAN的并发度一般不会很高。过度共享也会成为系统瓶颈。



Server failure -

&gt;

 no answer anymore from the server

  


服务器挂了。服务器没有响应了。  


流程:

  


Testing time

  


测试需要多长时间？



Development time

  


开发需要多长时间？



Team size

  


团队大小/

  


Budget

  


预算多少？



Code debt

  


代码量多少？



内存:



Out of memory -

&gt;

 kills process, go into swap 

&

 grind to a halt



内存用光了。进程被系统杀死，交换到Swap，最后被挂起。



Out of memory causing Disk Thrashing \(related to swap\)



内存用光，导致产生频繁的磁盘交换。



Memory library overhead



使用的库，占用过多的内存



Memory fragmentationIn Java requires GC pauses



内存分片，java中需要暂停GC。



In C, malloc's start taking forever

C中malloc始终是分配内存的开始。

