## 区块链之于AI：对抗数据寡头 {#activity-name}

_2018-03-06_

_白硕_

[待字闺中](https://mp.weixin.qq.com/s?__biz=MjM5ODIzNDQ3Mw==&mid=2649968033&idx=1&sn=5f3c1beec06b9c925f83c6f8cbf376ec&chksm=beca3da789bdb4b10c6be95a6b222dd6db286ff92258623a3f857db8a727421e2fb591cd5d41&mpshare=1&scene=1&srcid=03063WeyaDQ2ZNVlF3Depkg5##)

说到区块链，很多人很着急先给它戴一顶“**去中心化**”的帽子。我觉得，这个帽子戴不戴先不着急，我们还是来看一看区块链本质上到底是什么。受众中很多是技术大咖，谈论技术细节，大家接受起来都不成问题，但是从这个路子讲的人太多了，我不想重复。我想讲区块链的几个重要的特征，通过这几个重要的特征，我们争取对区块链有个顶层的观感，然后再看人工智能跟区块链有哪些结合点。  


  




时间不可逆：无法撤销、无法篡改、无法仿冒、无法乱序

  


**区块链的第一个特征，叫做时间不可逆。**物理学上讲熵单向增加的方向构成了我们所体验到的时间箭头，我们所体验到的过去和未来之间的不对称。一滴墨滴到一盆清水里，我们会看到墨滴在清水中扩散，直到整个盆里的水变浑浊、变灰、变黑。这个过程的逆向过程，也就是一盆黑水变灰变清，变回一滴墨刚滴到水中的原初形态，在我们这个服从热力学第二定律的世界里是看不到的，除非看倒着播放的视频。这个时间不可逆的特性，区块链上通过数据区块内部数据记录的分层哈希和数据块之间的哈希勾稽关系实现了出来，使得记录在数据区块上的数据无法撤销、无法篡改、无法仿冒、无法乱序。

  


**区块链的数据结构中有三个要紧的元数据，分别代表过去、现在和未来。**代表过去的是“**上一区块的哈希**”，它是上一区块内容的直接浓缩，也是到上一区块为止的所有历史数据的间接浓缩。代表现在的是本区块所有数据记录的梅克尔树根，它是本区块内所记录的所有数据的存在和他们之间的顺序关系。

  


代表未来的是**待定的随机数**，也就是被当做工作量证明的那个哈希函数反求的特解，这个随机数延伸到未来（下一区块）时，可以满足那个哈希函数取值的特定约束。也就是说，整个用这种方式勾连起来的区块，构成一部不可篡改的“历史”，单个的数据记录被有机地“缝合”进历史的长河，牵一发则动全身，要改一处记录就要对历史进行“分叉”，精准修改历史的局部，与为此付出的代价相比会得不偿失。这是由算法的数学性质保证的。**这个时间不可逆的特征，构成区块链的最基础的功能，就是可以被公开验证的“存证-定序”的功能。**

  


当然，这里说的“存证”，是数字化之后的“证”。在原子世界里的存证问题，比如饭店里盘中之鱼是不是服务员给你看的桶中之鱼，这不是区块链可以解决的。桥归桥路归路，原子世界的存证问题必须用原子世界的手段去解决，而不能用比特世界里的区块链技术去解决。

  




价值守恒：区块链的价值转移

  


**区块链的第二个特征，叫做价值守恒。**信息是不守恒的，简单的拷贝-粘贴，就可以几乎零成本地把信息复制任意多份。人类之间观念的沟通，输入者收获满满，输出者也没失去什么。把信息的这个特点用到极致，就是今天的互联网。但是，价值不同于信息。价值过去是实物的度量。“一手交钱、一手交货”，说的就是物权的转移伴随价值的逆向转移。而实物是满足守恒定律的。随着价值转移记录的数字化进程，金融机构里的价值很多都在用数字记账了，某种意义上，价值就体现为数字，而数字是很容易被修改的，单方面修改数字就会导致账不平，也就是价值不守恒。

  


为了保证数字记账的准确性，金融机构之间、金融机构与其客户之间要进行大量的对账操作，以建立金融机构的可信任性，金融机构内部也有大量的对账和校验流程，来防止内部人恶意篡改负载着价值语义的数据。但是，在一个互相没有信任基础的开放群体内，如何保证价值守恒，是长久以来没有解决的问题，区块链技术通过“**协议创新**”，在一定程度上解决了这个问题，从而使得本来用于传输不守恒的信息的互联网上，也能传输守恒的价值了。

  


要做到这一点，除了“存证-定序”功能之外，还要加上保证**价值守恒**的功能，也就是防止透支（无中生有）和双花（一款二用）的功能。这一点做到了，价值才能在群体中可信任地流通。这个价值守恒的特性，构成区块链的第二个功能，就是价值转移的功能。

  


现在虽然能够做到价值在一条区块链内部守恒，但是价值跨越区块链的边界从一条区块链进入另一条区块链的时候是否仍然守恒，价值传统的银行账户和区块链上的地址之间相互转移的时候是否依然守恒，个别区块链针对个别场景有了局部的进展，但这个**“跨链”价值转移**的问题从总体上说还没有统一的技术解决方案，一切都还在探索之中。

  


此外，价值转移的成立依赖于“**多数诚实记账者的共识见证**”，而价值转移的记录往往比较敏感，让这么多无关的记账者去见证，很多场景下是行不通的。如何在丝毫不透露真实的价值转移记录的前提下，仍能为见证者提供背书的足够理性依据，更扩展开来一点说，如何在多个经营主体各自守住自身数据主权边界的前提下达成数据共享合作，这也是一项重大的技术挑战，在技术上属于“**多方安全计算**”的研究领域。可喜的是，区块链的出现极大地刺激了“多方安全计算”的研究，这一领域中很多有价值的密码学成果正在借助各类区块链项目逐步落地。

  




让价值“飞一会儿”

**  
**

**区块链的第三个特征，叫做价值/信任状态的可编程特征。**有了前面两个特征，建立账本的基本技术条件就具备了，于是产生了**比特币**。比特币的整个数据就是一套分布式账本，上面记录了比特币从产生（发行）到转移（交易）的整个历史。但是，转账一结束，价值就停止流动了。这很大程度上制约了价值在数字化空间中存在的意义，数字化空间的真正威力并没有淋漓尽致地发挥出来。

  


如果我们能让价值在数字化的空间里“飞一会儿”，让这些飞翔着的价值跟外部发生的真实的事件、跟价值的所有者头脑中构想的真实的业务逻辑实现深度的耦合，那么数字化的价值就脱离了对账本和记账的简单模仿，而进入到发挥数字化价值特色、超越账本和记账的新阶段。

  


**这个能让价值“飞一会儿”的可编程的对象，就是“分布式应用（DApps）”，在一些区块链上体现为“智能合约”，在另一些区块链上体现为另外一些程序实体。**它们的共同特点，就是能够通过区块链所确认的代码，在区块链所确认的输入下，执行一定的“**业务逻辑**”，产生通过区块链所确认的输出。这个输出，包括了**价值的再分配**，也包括了**信任状态（比如特定的房产的抵押状态）的变更**。

  


分布式应用的出现，使得区块链所生产的对象从单纯的交换价值变为也包含使用价值，也使得在区块链上从事生产活动的角色从单纯的矿工变为也包括**码农**，支撑区块链运作的资源则从单纯的算力变为也包含**智力**。这是一个重大的变化，**这个变化导致区块链逐渐向所谓的“数字经济体”转化**，其中的交换价值和使用价值可以实现闭环运行，其中的使用价值可以和真实世界、和人们的社会生活实现广泛的对接。当这个“数字经济体”达到很大体量的时候，它对社会生活各方面的影响是不可忽视的。当然，分布式应用也带来了一系列全新的问题。要解决这些问题，不可能技术单兵突进，而必须各方面配套。前几年陆续出现的分布式应用疑似“事故”，就给区块链业界以深刻的教训。

  




自带商业模式：自带激励和自带营销

  


**区块链的第四个特征，叫做自带商业模式。**这里“自带商业模式”的含义是，**区块链的技术架构和技术设定隐含地假设了一种全新的生产关系**。这种生产关系有能力在区块链内部产生生产和销售的动力，也就是通常所说的“自带激励”和“自带营销”。

  


从前面三个特征可以看出，区块链的主要“卖点”就是**信任**，而面对一个黑盒子，是不能产生信任的。所以，有影响的区块链几乎无一例外地采用了**开源策略**。开源是双刃剑。它既倡导了技术的开放共享，提高了创新技术的采纳效率，但是也降低了门槛，使得区块链及分布式应用的开发人员工作在没有护城河保护的危机之中。所以，**自带激励**，不仅对于“矿工”是必要的，对于“码农”来说更加必要。打赏这一“工”一“农”，是数字经济体良性发展不可缺少的润滑剂，也是区块链自带激励的初衷。光有生产者还不够，区块链及其上的分布式应用未来的拓展，也要靠其开发初期对项目有信心、有到位理解的人士为之奔走相告，未来使用权的承销也是需要给予激励的，而区块链把未来使用权这种权益也通过区块链来承载，就成了“**自带营销**”。

  


对于区块链“自带商业模式”这个特征，需要全面理解和慎重对待。无约束、无监管的“自带营销”是很容易发展成为变相集资、变相传销、变相从事证券业务的，这在各国的金融相关法律中都有不同程度的管制。但是，纯使用权性质的“自带商业模式”，应该等同于商品众筹，这在多数国家包括我国都是合法的。如何对不同性质的“自带商业模式”实行差异化、精细化的监管，鼓励正当的“自带商业模式”的区块链项目落地并对接社会生活，是需要我们认真思考的。

  




每一项技术的砖不新鲜，但它搭的“楼”却有真正的创新和创意

  


讲完了区块链的这四个重要的特征，相信大家已经初步建立起了对区块链的顶层观感。我没有光讲好话，每一个特征我都讲了它的问题和挑战。但是大家可以看出，区块链这东西，真的是一种奇葩的存在。你可以说它使用的每一项技术“砖块”都不是新鲜的，但是它搭建起来的“楼”却有真正的创新和创意在里面。它的核心技术涉及了密码学、分布式系统、对等网络和算法博弈论。

  


当然，关于区块链如何“威胁”甚至“颠覆”传统金融的各种说法也纷纷出笼，“去中心化”这样本来属于一部分激进无政府主义的区块链业界人士的理念，也被整体加到了“区块链”这个整体领域的头上，这是不准确的。区块链是一项技术，它可以有中心化的应用，也可以有去中心化的应用，甚至可以是部分中心化部分去中心化的组合。但是总体上，由于信任的物质基础、信任的抓手和信任的路径有所改变，信任的结构必然随之有所改变。对于这种改变，既要保持足够的警觉，又要具体情况具体分析，不宜一概排斥也不宜不问场景照搬照抄。  


  


区块链业界根据所建区块链的部署特点和运营特点，分成了所谓的**“币圈”和“链圈”**。大体上说，“币圈”必须是平台、代币、社区三位一体，一个都不能少的，“链圈”则只要有“平台”就够了。“币圈”更多是2C的打法，“链圈”更多是2B的打法。“币圈”更多是公链部署，“链圈”更多是私链或联盟链部署。但不可否认的是，区块链领域的大多数重要技术创新来自“币圈”。而“链圈”如果不能很好地解决激励问题的话，能否可持续发展都会成为问题。

  




区块链+人工智能：数据寡头大敌

  


说到这儿，我们回头说人工智能。我本人是人工智能科班出身的博士，经历了人工智能的低潮时期，但一直坚持着对人工智能当中最有挑战性的“自然语言处理”领域的研究兴趣，大家在新智元的“语义计算”分群里可以看到我关于NLP的思考和实践。我在新智元也做过人工智能领域的数次专题分享。我在学术界时做过安全协议的形式化验证相关工作，在信息产业部门做过管理信息安全技术的官员，在上交所主持过超过20万tps的证券交易系统的升级换代工作，信息安全+金融+IT的综合背景使我较早地接触了区块链技术。目前最火的两个领域——人工智能和区块链的完整知识结构长在同一个人的脑子里，这是不常见的，本人有幸阴差阳错成为其中一个。所以，人工智能遇见区块链会产生怎样的火花，对有些朋友来说是新问题，对本人来说则是一个每天都在刷新认知的习以为常的问题。我想从两个方面来谈人工智能与区块链的关系。

  


**第一个方面是：人工智能能为区块链做什么？**

  


我的回答是，人工智能用于区块链，这是一片广阔的蓝海。从全球范围看，区块链领域卷入了太多的人，其中大部分人有对新事物的热情，有尝试新机会的冲动，但是他们能够获得的有价值的服务太少太少。为区块链领域提供智能化的培训和教育，提高被卷入区块链领域的人们的专业素质和识别能力，是最迫切、最有情怀的需求。利用人工智能模型赋能虚拟货币的监管，也会产生巨大的社会效益。在这两者之间，有着广阔的空白地带和丰富的想象空间。虽然中国不让ICO、不让交易虚拟货币了，但世界上还有三分之二受苦受难的阶级兄弟，中国还有一些人哭着喊着到境外去干这事儿。人工智能如果能够帮助他们脱离苦海，功莫大焉。

  


**第二个方面是：区块链能为人工智能做什么。**

  


我认为，人工智能火起来的一个最大的动因，就是**数据的聚集和数据寡头的形成**。智能都是数据喂出来的，没有数据的聚集，就失去了产生智能的物质基础。但，现实中我们已经切切实实感受到了数据寡头对我们的威胁：每个个人都被一个又一个的数据寡头们“画像”，他们知道我们知道得太多了。小公司小企业则时刻面临从数据上要么被边缘化、孤岛化，要么被通吃的威胁。如能够做到在数据不离开自己的主权边界的情况下产生和数据聚集情况下同等效果的智能，小公司就会有结成“联盟化虚拟数据寡头”这个活路，个人数据也有望获得更好的保障。这个问题，在人工神经网络的语境下，转化为“同态神经网络”的问题；在知识图谱的语境下，转化为“同态知识图谱”的问题。无论哪一个问题，都有巨大的商业价值和技术上的挑战性。解决了这类问题，也防止了人工智能沦为仅仅是寡头们的专属工具，把它真正还给人民大众。

  


总而言之，人工智能和区块链的深度结合，想象空间是无比巨大的。

