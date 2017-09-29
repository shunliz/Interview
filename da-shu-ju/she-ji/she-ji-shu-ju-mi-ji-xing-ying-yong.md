# Designing Data-Intensive Applications The Big Ideas Behind Reliable, Scalable, and Maintainable Systems {#firstHeading}

## 目录

* [1可靠性、可伸展性、可维护性](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E5.8F.AF.E9.9D.A0.E6.80.A7.E3.80.81.E5.8F.AF.E4.BC.B8.E5.B1.95.E6.80.A7.E3.80.81.E5.8F.AF.E7.BB.B4.E6.8A.A4.E6.80.A7)
* [2数据模型与查询语言](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E6.95.B0.E6.8D.AE.E6.A8.A1.E5.9E.8B.E4.B8.8E.E6.9F.A5.E8.AF.A2.E8.AF.AD.E8.A8.80)
* [3存储与检索](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E5.AD.98.E5.82.A8.E4.B8.8E.E6.A3.80.E7.B4.A2)
* [4编码与演进](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E7.BC.96.E7.A0.81.E4.B8.8E.E6.BC.94.E8.BF.9B)
* [5复制](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E5.A4.8D.E5.88.B6)
* [6分区](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E5.88.86.E5.8C.BA)
* [7事务](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E4.BA.8B.E5.8A.A1)
* [8分布式系统的麻烦](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E5.88.86.E5.B8.83.E5.BC.8F.E7.B3.BB.E7.BB.9F.E7.9A.84.E9.BA.BB.E7.83.A6)
* [9一致性与选举](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E4.B8.80.E8.87.B4.E6.80.A7.E4.B8.8E.E9.80.89.E4.B8.BE)
* [10批处理](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E6.89.B9.E5.A4.84.E7.90.86)
* [11流处理](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E6.B5.81.E5.A4.84.E7.90.86)
* [12数据系统的未来](http://localhost/mediawiki-1.25.3/index.php/Designing_Data-Intensive_Applications_The_Big_Ideas_Behind_Reliable,_Scalable,_and_Maintainable_Systems#.E6.95.B0.E6.8D.AE.E7.B3.BB.E7.BB.9F.E7.9A.84.E6.9C.AA.E6.9D.A5)

## 可靠性、可伸展性、可维护性

## 数据模型与查询语言

1. schema更新：
   1. 关系SQL：~ 静态语言
   2. NoSQL：动态类型语言
      1. Document型：MongoDB的BSON --
         &gt;
          整个重写替换/部分更新？--
         &gt;
          增量快照？
2. grouping related data --
   &gt;
    locality
   1. Spanner的“嵌套表”
   2. Oracle的“多表索引聚类表”
3. 文档型与关系型的融合：RethinkDB（列向）--
   &gt;
    关系-like joins？
4. MongoDB 2.2+ 聚集管道，声明式的map-reduce
5. 图
   1. 属性图：Neo4j、titan、InfiniteGraph
   2. triple-store：Datomic、AllegroGraph
6. 3' 声明式语言：Cypher，SPARQL（基于RDF），Datalog（Prolog/LISP的变体,学术型）
7. vs CODASYL（网络模型）

## 存储与检索

1. log --
   &gt;
    segments with hash index --
   &gt;
    compact去重
   * Range查询不够高效
2. SSTable --
   &gt;
    memtable（key-value） + log日志
   * 思想来自于BigTable，如LevelDB、RocksDB
3. LSM-tree
   * used by fulltext index（倒排的存储，Lucene）
4. Bloom-filter：早期排除不存在的key（？但不能排除的情况下还是一样的）
5. compact 
   &
    merge的时机：
   1. size-tierd
   2. leveled
6. B-tree：标准索引实现
   1. Making reliable：WAL（redo log）
   2. 写操作对SSD不友好？
   3. 并发控制：latch锁
   4. 优化\*
7. LSM-树后台压缩影响性能，不如B-树可预测（实质是均摊了）
8. 其他索引
   1. secondary（Map
      &lt;
      K, List
      &lt;
      V
      &gt;
      &gt;
      ）
   2. clustered（value in index）
   3. 多列组合
   4. 全文与fuzzy
9. 内存数据库
10. 数据仓库：分析查询
    1. 产商产品：Teradata, Vertica, SAP HANA, ParAccel; Aws RedShift; SQL-on-Hadoop（Hive, SparkSQL, Impala, Presto, Tajo, 基于Dremel的Drill）'
    2. 星模式：事实表（events log？）+ 维度表（进一步导出的？）
       1. 雪花
11. 列向存储
    1. 列压缩：位图编码（enum--
       &gt;
       bool），适用于IN \(...\)类的查询？
    2. 多个sort orders？
12. 聚集：Data Cube 
    &
     物化视图\*

## 编码与演进

1. JSON，Bin（Base64？）
2. BSON：扩展了datatypes
3. Thrift 
   &
    protobuf（2007年开源）
   1. 无field name字符串，而是tags（枚举整数）
4. Avro：len头部 + utf8字节
   1. reader 
      &
       writer's schema
   2. 模式演化规则：field有default value
5. ASN.1：如DER
6. 分布式actors：Akka、Orleans、Erlang OTP

## 复制

1. replication lag下的一致性模型
   1. Read-after-write consistency
   2. Monotonic reads
   3. Consistent prefix reads（因果序）
2. Leader-less Dynamo-style（不保证写的顺序？但是如果有应用级的hash路由呢？如果所有数据潜在可关联如社交网络，就不适用了...）
   1. client发起并行读，取version最高的？
      1. 然后client回写更新版本过期的
      2. 或服务器Anti-entropy process后台同步？
   2. Quorums：w + r 
      &gt;
       n
   3. Quorums方法的局限性（略）
   4. Monitoring staleness
      1. 度量replication lag
   5. Sloppy Quorums and Hinted Handoff
      1. 增加写的可用性，但是读的可靠性下降
      2. Sloppy quorums are optional in all common Dynamo implementations. In Riak they are enabled by default, and in Cassandra and Voldemort they are disabled by default（很奇怪的不同设计？）
   6. Detecting Concurrent Writes（多个clients同时写同一个key）
      1. Last write wins \(discarding concurrent writes\) 应用级别的版本化？
      2. The “happens-before” relationship and concurrency（编程语言的内存模型里的术语）
         1. 两者不相关就是并发，不需要全局时间戳（网络～光）
      3. Capturing the happens-before relationship
         1. 版本号：写之前必须读，获取当前的最新版本号（全局快照），而客户端写之前自己负责处理merge
         2. 妈的这不就是SVN／Git的通常使用模型嘛
      4. Merging concurrently written values
         1. Such a \`deletion marker\` is known as a \`tombstone\`.
      5. Version vectors\(自动branching...\)

## 分区

1. 分区策略
   1. by Key Range：数据热点问题
   2. by Hash of Key：不支持key range查询
      1. Consistent Hashing：术语“一致性”指的是rebalancing方法
      2. Cassandra：compound primary key + 仅hash第一个key（Hash of Range？）
   3. 分区不能处理对同一个key对大规模并发读写：Skewed Workloads and Relieving Hot Spots
2. &
    二级索引（文档数据库中的文档属性）
   1. document-based + local index：scatter／gather read
      1. 理论上，如果不是用文档id直接作为主key，而是用这些二级索引属性的组合hash作为key的话，可以避免
   2. term-based：全局的index及其partition
      1. 写操作更复杂了：所有数据库都不支持“分布式事务”？
3. Rebalancing
   1. 不要使用 % N方法，减少data move的开销！
   2. 解决方法1：一个物理node上分配多个partition
      1. 一开始就固定住总的partition数？node与partition的关联需要手工维护？
      2. how to 选择正确的总partition数？？
   3. Dynamic partitioning
4. Request Routing
   1. routing tier：acts as a partition-aware load balancer
   2. ZooKeeper：keep routing info up-to-ate

## 事务

1. ACID
   1. The word \`consistency\` is terribly overloaded（必须由应用来保证？）
   2. Isolation
      1. serializable：性能问题，Oracle甚至没有实现它！
      2. snapshot（MVCC）
   3. 作者的批评：... such as Rails’s ActiveRecord and Django don’t retry aborted transactions
2. Weak Isolation Levels
   1. Read Committed：无脏读、无脏写
      1. dirty write：一个事务的写覆盖了另一个事务未提交的读
      2. 实现：
         1. 行级锁
   2. Snapshot Isolation and Repeatable Read
      1. 不能容忍临时不一致的情况：
         1. Backups
         2. Analytic queries and integrity checks
      2. 实现：
         1. 关键原则：readers never block writers, and writers never block readers
         2. txid：每一行关联created\_by、deleted\_by（额外的GC进程），update转换为delete + create？
         3. 对象可见性规则
      3. Indexes：append-only B-tree
      4. ... As a result, nobody really knows what repeatable read means.
   3. Preventing Lost Updates
      1. 2个并发的read-modify-write
      2. 原子写
      3. 应用侧的Explicit locking：SELECT ... FOR UPDATE
      4. 自动检测并abort
      5. Compare-and-set
      6. 副本问题：Conflict resolution and replication（原子操作需要是可交换的）
   4. Write Skew and Phantoms
      1. 2个事务快照读同样的对象，然后分别进行写更新操作（并发带来的问题）
      2. =
         &gt;
          SELECT ... FOR UPDATE（但不能处理后续写不是之前快照读的对象的情况）
      3. Materializing conflicts
   5. Serializability
      1. Literally executing transactions in a serial order（内存数据库？Redis）
         1. systems with single-threaded serial transaction processing don’t allow interactive multi-statement transactions.（存储过程？）
            * 作者这里吹嘘了VoltDB一把～
      2. 2PL
         1. 2PL不是2PC
         2. 共享锁 -
            &gt;
             排它锁
            1. deadlock
         3. Predicate locks（锁住一个‘搜索条件’）
            1. Index-range locks
      3. Optimistic concurrency control techniques such as serializable snapshot isolation（SSI）
         1. 乐观的并发控制技术在高Contention下有性能问题？
         2. 检测outdated premise
            1. Detecting stale MVCC reads（？？？）
               1. 这里的问题是：SSI怎么知道哪些数据属于快照创建时uncommited writes同时commit时已被修改？
               2. the database needs to track when a transaction ignores another transaction’s writes due to MVCC visibility rules（？？？）
            2. Detecting writes that affect prior reads

## 分布式系统的麻烦

1. Monotonic Versus Time-of-Day Clocks
2. confidence interval
   1. Google’s TrueTime API in Spanner：\[earliest, latest\]
3. Knowledge, Truth, and Lies
   1. The Truth Is Defined by the Majority：但是quorum的quorum，这会导致独裁吧？
   2. Fencing tokens
   3. Byzantine Faults
      1. 太空辐射导致的硬件错误？一个bit的翻转？
      2. Most Byzantine fault-tolerant algorithms require a supermajority of more than 2/3 of the nodes to be functioning correctly
   4. system model（fault的模型），如timing：
      1. Synchronous model：有上界？不太现实
      2. Partially synchronous model
      3. Asynchronous model
   5. Safety and liveness

## 一致性与选举

1. stronger consistency: worse performance or less fault-tolerant
2. Linearizability
   1. linearizability is a
      **recency guarantee**
      （一旦x read到新值，则所有后续read都应该独到新值）
   2. cas：只有当x值为old的时候更新到new，否则fail
   3. vs Serializability
      1. 序列化是事务的隔离级别属性，而线性化是对register的读写的recency保证，它不能防止write skew问题
      2. serializable snapshot isolation（SSI）is not linearizable
   4. Uniqueness constraints
      1. 原文：Strictly speaking, ZooKeeper and etcd provide linearizable writes, but reads may be stale ...
   5. 实现：
      1. Multi-leader replication \(not linearizable\)
      2. Single-leader replication \(potentially linearizable\) + Consensus algorithms \(linearizable\)
      3. Leaderless replication \(probably not linearizable\)
3. CAP
   1. Consistency, Availability, Partition tolerance: pick 2 out of 3
   2. 网络分区是一种fault，不是你可以自由选择的（没得选择）
4. Ordering Guarantees
   1. If a system obeys the ordering imposed by causality, we say that it is
      **causally consistent**
      .
   2. linearizability是全序，而causality是偏序关系（有没有可能是“线性化+可水平扩展”？）
   3. Sequence Number Ordering
      1. 非single leader的情况：Noncausal sequence number generators
         1. 确保全局uid是可行的（预分配uid block）；使用高解析的全局时钟。缺点：not consistent with causality
   4. Lamport timestamps：\(counter, node ID\)
      1. every node and every client keeps track of the maximum counter value it has seen so far, and includes that maximum on every request.
      2. Lamport timestamps are sometimes confused with version vectors（enforce a total ordering）
   5. Timestamp ordering is not sufficient（两个用户同时注册以抢占一个userid？这个问题有点病态的说）
      1. This idea of knowing when your total order is finalized is captured in the topic of “total order broadcast”.——有点新鲜，我怎么没听说过？
   6. 全序广播（原子广播）
      1. 算法需要确保safety属性：
         1. Reliable delivery（消息必须可靠传播，保证每个node都接受到）
         2. Totally ordered delivery（消息的node接受顺序相同）——缺点：这个顺序不能动态修改？？
      2. Implementing linearizable storage using TOB
         1. 实现“线性化CAS操作”：... Read the log, and wait for the message you appended to be delivered back to you（？？？这里的描述有点含糊啊）
            1. 注释： If you don’t wait, but acknowledge the write immediately after it has been enqueued, you get something similar to the memory consistency model of multi-core x86 processors \[43\]. That model is neither linearizable nor sequentially consistent.
5. 分布式事务与Consensus
   1. FLP：The Impossibility of Consensus（in the asynchronous system model）
   2. 2PC与原子提交
      1. coordinator：prepare/commit
      2. A system of promises: a bit more details
         1. 全局唯一的transaction ID
         2. 各个参与者的读写操作都关联到此事务id
         3. By replying “yes” to the coordinator, the node promises to commit the transaction without error if requested. In other words, the participant surrenders the right to abort the transaction, but without actually committing it（但是这个承诺可靠吗？由于不可抗的外部灾难导致硬件故障呢？？）
         4. Once the coordinator’s decision has been written to disk, the commit or abort request is sent to all participants. If this request fails or times out, the coordinator must retry forever until it succeeds.（这玩意儿也能称得上是算法吗？？？扯淡）
      3. 3PC：blocking --
         &gt;
          nonblocking
         1. ！3PC assumes a network with bounded delay and nodes with bounded response times
         2. **perfect failure detector**
            （但是reliable基本上不可能，因网络超时在分布式系统里等同于节点失败）
   3. 实践中的分布式事务
      1. internal（所有node运行同样的协议）vs heterogeneous
      2. Exactly-once message processing
      3. XA is not a network protocol—it is merely a C API for interfacing with a transaction coordinator.
         1. 接受repare通知相当于在XA Driver的C API上注册callback？？？靠
      4. Holding locks while in doubt
      5. Recovering from coordinator failure（commit log意外丢失导致不能决定提交/abort）
         1. The only way out is for an administrator to manually decide whether to commit or roll back the transactions.（人工干预）
   4. Fault-Tolerant Consensus
      1. 算法属性:
         1. Uniform agreement：No two nodes decide differently.
         2. Integrity：No node decides twice.
         3. Validity：If a node decides value v, then v was proposed by some node.
         4. Termination：Every node that does not crash eventually decides some value.
      2. The best-known fault-tolerant consensus：Viewstamped Replication \(VSR\), Paxos, Raft, and Zab
      3. Epoch numbering and quorums（每个epoch周期内leader唯一，但不需要是同一个）
         1. ... can't decide by itself, Instead, it must collect votes from a quorum of nodes ...
         2. 2轮选举: 一次 a leader, 一次 leader’s proposal.
      4. Limitations of consensus
         1. ... designing algorithms that are more
            **robust to unreliable networks**
            is still an open research problem
   5. Membership and Coordination Service
      1. HBase, Hadoop YARN, OpenStack Nova, and Kafka all rely on ZooKeeper running in the background.
      2. ZooKeeper is modeled after Google’s Chubby lock service, implementing not only total order broadcast：
         1. Linearizable atomic operations
         2. Total ordering of operations（zxid）
         3. Failure detection
         4. Change notifications
      3. ZooKeeper, etcd, and Consul are also often used for
         **service discovery**
      4. read-only caching replicas：只缓存consensus算法的结果，不参与投票

注意，本章没有描述consensus算法的细节（如Paxos或Raft）

## 批处理

1. MapReduce workflows（通过硬编码的HDFS ouput路径？）
   1. 一个单独的MapReduce job无法有效处理Top-N问题？
2. Reduce-Side Joins and Grouping
   1. Sort-merge joins
   2. Bringing related data together in the same place
   3. GROUP BY（聚集查询）
   4. Handling skew
      1. hot keys问题：
         1. **skewed join**
            in Pig（预采样处理？需要完全复制hot key关联的其他数据）
         2. **sharded join**
            in Crunch：类似的，但不是预采样，而是要求人工指定（？）
         3. Hive：mapper side join
3. Map-Side Joins
   1. Broadcast hash joins：大数据集连接小数据集（后者可完全加载到内存hashtable）
   2. Partitioned hash joins（如果连接双方的key都依据相同的规则partition的话）
   3. Map-side merge joins
4. The Output of Batch Workflows
   1. Key-value stores as batch process output
   2. immutable input：buggy code can be corrected and re-run
   3. using more structured file formats: Avro, Parquet
5. Hadoop vs MPP
   1. 优势：可以快速地载入到HDFS... \(MPP系统需要预先做结构化建模\)
   2. 允许Overcommitting resources
      1. At Google, a MapReduce task that runs for an hour has an approximately 5% risk of being terminated to make space for a higher-priority process
6. Beyond MapReduce
   1. “simple abstraction on top of a distributed filesystem”：理解容易，但是具体的job实现并不简单（MapReduce是很低级的API）
   2. Materialization of Intermediate State
      1. MapReduce的状态物化的缺点（vs Unix Pipes）：略
      2. Dataflow engines：Spark、Tez、Flink
         1. 避免昂贵的排序？
         2. \*JVM进程可以被重用（Hadoop里面每个task都要重启新的）
      3. Dataflow系统的Fault tolerance：
         1. 如何描述ancestry数据输入？（Spark RDD）
         2. make operators deterministic
   3. Graphs and Iterative Processing
      1. PageRank
      2. Pregel处理模型：bulk synchronous parallel \(BSP\), vertex之间相互发送消息
         1. Pregel model guarantees that all messages sent in one iteration are delivered in the next iteration, the prior iteration must completely finish, and all of its messages must be copied over the network, before the next one can start.
         2. This fault tolerance is achieved by periodically
            **checkpointing**
            the state of all vertices at the end of an iteration
         3. 问题：机器之间的通信开销比较大
   4. High-Level APIs and Languages
      1. UAF：Spark generates JVM bytecode and Impala uses LLVM to generate native code for these inner loops
      2. Specialization for different domains：Mahout、Spatial-kNN

## 流处理

1. Transmitting Event Streams
   1. Messaging Systems：pub/sub
      1. Message broker（消息队列）：允许consumer暂时离线
      2. Multiple consumers：fan-out
      3. Acknowledgments and redelivery
         1. the combination of load balancing with redelivery inevitably leads to messages being reordered
2. Partitioned Logs
   1. log-based message brokers：Apache Kafka
   2. 典型的磁盘大小：6TB，写速度450MB/s，则大约11小时写满
3. Databases and Streams
   1. Change Data Capture（CDC）：捕获对数据库的修改，作为流输出（需要解析redo log/WAL）
      1. Initial snapshot
      2. Log compaction
      3. API support for change streams
   2. Event Sourcing（来自于DDD社区）：从高层建模？
      1. Deriving current state from the event log
      2. Commands and events
   3. State, Streams, and Immutability
      1. 分离数据读写的形式：command query responsibility segregation \(CQRS\)
      2. append another event to the log，假装数据被“删除”了？哈哈哈
4. Processing Streams
   1. Complex event processing \(CEP\)：查询（声明式的模式搜索）变成持久的
   2. Stream analytics：window、概率算法
   3. Maintaining materialized views
   4. Search on streams
   5. vs 其他Messaging/RPC（如actors模型）
      1. 融合：Apache Storm的distributed RPC特性
   6. Event time versus processing time
   7. Types of windows（感觉这里的讨论有点无趣）
      1. Tumbling window
      2. Hopping window：允许重叠
      3. Sliding window
      4. Session window
   8. Stream Joins（感觉这里的讨论有点无趣）
      1. Stream-stream join \(window join\) 略
      2. Stream-table join \(stream enrichment\)
      3. Table-table join \(materialized view maintenance\)
   9. Time-dependence of joins
5. Fault Tolerance
   1. Microbatching and checkpointing
   2. Atomic commit revisited（‘exactly-once’）
   3. Idempotence
   4. Rebuilding state after a failure

这部分内容大部分都很扯淡（空谈），感觉有必要仔细看看Kafka的设计？？

## 数据系统的未来

1. 数据集成
   1. Schema Migrations on Railways（渐进迁移！）
   2. The lambda architecture：immutable append-only + stream + batch？
2. Unbundling Databases
   1. Transactions within a single storage or stream processing system are feasible, but when data crosses the boundary between different technologies, I believe that an asynchronous event log with
      **idempotent writes**
      is a much more robust and practical approach
   2. Observing Derived State
      1. Taken together, the write path and the read path encompass the whole journey of the data ...
      2. Materialized views and caching
      3. Stateful, offline-capable clients
         1. **offline-first**
         2. In particular, we can think of the on-device state as a cache of state on the server. The pixels on the screen are a materialized view onto model objects in the client app; the model objects are a local replica of state in a remote datacenter
      4. End-to-end event streams
      5. Reads are events too（有点去中心化的感觉？作者的奇思妙想？）
      6. Multi-partition data processing
3. Aiming for Correctness
   1. “exactly once”, duplicate supression, operation identifiers
   2. The end-to-end argument（需要使用一个端到端的事务ID吗？）
   3. Applying end-to-end thinking in data systems
      1. 作者这里啥也没说。重新探寻“事务”的抽象是否合适？
   4. Enforcing Constraints
      1. 不需要multi-partition atomic commit：基于event的log，附带一个“Request ID”即可（可trace
         &
         可audit）
         1. 备注：event log实际上就用在Bitcoint这种分布式P2P去中心化的应用中（不过Request ID机制在比特币的交易模型中并不明显，因为块链要求全局线性化同步。。。）
         2. 这里还有一个问题：Request ID到底由谁来分配？应该是app client side，而不是服务器前端（这不就是DDD嘛）
            1. 实际上，并不需要端到端的Request ID，只要保证领域转换（DDD术语）可以正确映射领域对象id即可
   5. 一致性：Timeliness and Integrity
      1. Timeliness means ensuring that users observe the system in an up-to-date state.
      2. Integrity means absence of corruption; i.e., no data loss, and no contradictory or false data
      3. event-based dataflow systems解耦这2者？（由于event处理是异步的，sender不能立即看到结果）
         1. Passing a client-generated request ID through all these levels of processing, enabling end-to-end duplicate suppression and idempotence
      4. Similarly, many airlines overbook airplanes in the expectation that some passengers will miss their flight,...（数据模型的一致性有时候被业务模型直接破坏，引入了补偿事务/系统外的人工干预）
         1. **apology**
      5. Coordination-avoiding data systems
         1. 可部署到“跨数据中心、multi-leader配置、异步复制”的环境，一致性总是可以临时violated，事后再修复即可（通过end-to-end的回溯log可以做到这一点）
   6. Trust, but Verify
      1. “system model”：计算机不会犯错误，磁盘fsync的数据不会丢失，...
         1. but：硬件bit flip错误，“rowhammer”攻击
      2. If you want to be sure that your data is still there, you have to actually read it and check.
      3. Designing for
         **auditability**
         （可审查性）
      4. Tools for auditable data systems
         1. use cryptographic tools
         2. \(作者的看法\)
            **proof of work**
            很浪费？
         3. Merkle trees
            1. certificate transparency
4. 做正确的事情
   1. A technology is not good or bad in itself—what matters is how it is used and how it affects people.
   2. 例如：Predictive Analytics
      1. Bias and discrimination
      2. Responsibility and accountability
      3. Feedback loops
   3. Privacy and Tracking（备注：尤其是医疗系统）
      1. Surveillance
      2. Consent and freedom of choice（
         **de facto mandatory**
         ）
      3. Having
         **privacy**
         does not mean keeping everything secret; it means having the freedom to choose which things to reveal to whom, what to make public, and what to keep secret.
         1. 备注：隐私问题的本质在于，数据本身一旦存储下来，理论是可以无限制的保存下去的（除非硬件失效），而人的大脑记忆可以选择性遗忘
      4. Legislation and self-regulation



