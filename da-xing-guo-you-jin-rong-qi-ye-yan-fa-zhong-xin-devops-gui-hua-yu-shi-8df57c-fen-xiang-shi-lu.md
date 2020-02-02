以下文章来源于DBAplus社群，作者姚元庆

[![](http://wx.qlogo.cn/mmhead/Q3auHgzwzM5Xc1WG9yM88sYp9Rj7egU0Jtp3fYQQk7PtOAGv0kVZuA/0 "DBAplus社群")**DBAplus社群**围绕Database、Bigdata、AiOps的企业级专业社群。顶级大咖、技术干货，每天精品原创文章推送，每周线上技术分享，每月线下技术沙龙，受众20W+。](https://mp.weixin.qq.com/s?__biz=MzA5NzU3Njc5Mw==&mid=2651205095&idx=1&sn=fa108677f2848b5c55059b6b4e1a9e4a&chksm=8b6c3fd1bc1bb6c7752ec0461dbd5914d8d92e1b44529b8fbd82efb99bf2d1f8b3dba477669b&scene=0&xtrack=1&key=5dcdf35eccaf36f715dc7c72377b08a7d1b41e8195b9ba824f6b22c944ed64efc1a4ed3b0cd66237a844d8c4b4fae77cf3afa60a687a87a92d9c4f3751c11a9e11809e7d2989406b7d305b6c466c231f&ascene=14&uin=MTM1OTI2NzE0Mw%3D%3D&devicetype=Windows+10&version=6208006f&lang=zh_CN&exportkey=AelI8bJ5VKEt0KOO4%2BEfrW8%3D&pass_ticket=43efzXQOUw0Q3b32bbSFe9BhLOyTS4nvrYP0H67IlDdRe20YrLPCefWX86sJ0g3Y&winzoom=1#)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/tibrg3AoIJTuFebPiawmA9aemwZiaYrJicziaka1fvvFunLVr93uOUYYbAjiaMVsKot5ZGnft04ibEOoU0h94jU3oxesw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


> 本文根据〖2019 DAMS中国数据智能管理峰会〗现场演讲内容整理而成。
>
> 原文首发于公众号：DBAplus社群。点击“阅读原文”领取PPT
>
> 分享嘉宾：姚元庆，中国农业银行研发中心资深专员，具有十余年金融业软件研发、项目管理经验。长期致力于组织级项目管理、PMO建设、敏捷研发规划，以及作为教练为项目提供支持等工作。

  


大家好，今天与大家分享的内容是

「DevOps在国有大型商业银行的规划与实践」，从国有大型商业银行视角看一下DevOps怎么样规划与实践。

分享内容大致是这样的：

1、目标和背景

2、体系架构

3、三条主线，即：

工具、流程、规范

4、总结

  


**一、目标和背景**

![](https://mmbiz.qpic.cn/mmbiz_png/3MRbgjUiaA2XictGc0H4D2icx1ZFPS2ia92qIXp9ywTPagWCoKRVvc7TbrGWo7GPxdww6hy8UKt1s0oxBClqaTjydw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


  


![](https://mmbiz.qpic.cn/sz_mmbiz_png/tibrg3AoIJTuFebPiawmA9aemwZiaYrJicziaV7sLjQC3f4ZiaOZrib2GjjZamDeHmlQ6LMCuWBhqrHGJnP77BJakheMQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在去年下半年和今年上半年，我行提出了

数字化转型战略

。今年上半年的机构体制改革已经完成了，业务和科技部门在组织结构上进行了相关调整。

金融行业在研发过程中，经常提到的是

双模研发

，其中在核心系统上还是要用原来传统的方式来进行研发。

在业务角度，核心系统不会有那么大的变化，相对变化比较大一般都是掌银、网银及新兴业务方面。

所以，在核心系统上采用稳态研发、瀑布式的或者接近瀑布式的方式是一个非常理智的选择。

而在其他方面，比如说网银掌银，市场变化压力非常大，如果不快起来，业务也会让你快起来，企业也会让你快，还要保持四平八稳，那是不想干了，是吧！

今年6月份，

研发中心与信通院在DevOps方面进行共建。

我们

DevOps方面的目标可以简要概括为

**“1、3、5”**

：一个平台（DevOps），连接三个角色（开发、测试、运维），打通五个环节（需求、开发、测试、部署、运维）。

如此构建农业银行研发中心的DevOps体系。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  


**二、体系框架**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  


  


体系框架方面，与大家说体系框架之前，把我最深刻一个体会与大家分享：那就是“零”。

“零”是什么呢？其实对DevOps内容来说，大家觉得DevOps是干什么的？对于企业来说：DevOps也好、敏捷也好、之前的CMMI也好，都只是企业实现目标的一个工具和手段。

有段时间CMMI、DevOps或者敏捷，相关内容在企业进行推销的时候，如果

Get到企业的管理诉求、解决企业的痛点问题，

其实叫DevOps还是什么都无所谓，尤其对大企业来讲，

这个是最最重要一点。

还有一点我需要跟大家特别掏心窝的说一下，DevOps是什么？在学术和交流角度务必要清楚；但是，在企业角度最重要的是它能给企业带来什么价值。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

DevOps是什么？拿这个图来说。大家看像是一个房子。DevOps就是装修队如何来装修你的房子。

DevOps在传播过程中，首要提到的是拆墙（如：拆除研发与运维的墙、业务与技术的墙，当然也有企业与用户的墙）。让大家能更好地沟通和交流，快速实现价值的交付。

这两年，我们科技圈的Dev和Ops是不够，视野太狭窄了。其实至少要到科技和业务，拿银行的词就是

通过

“

业技融合”来快速响应市场的需求。

以下几点很重要：有的企业能拆墙，有的企业只能从墙上开一扇门，有的企业最多能开一扇窗，有的企业可能只能钻一个孔。

对于实际执行者，需要考虑一下，适合企业现状的是拆墙、建门、开窗还是打孔，这个是最关键的一个点，否则大家就是在聊DevOps，聊DevOps而不是在做DevOps。

在做DevOps时一定要记住：你是在干嘛？拆墙、开门、开窗、还是打孔。

如果孔都打不了，那DevOps就展示不要想了，先等等或尝试推动。

我们刚开始做敏捷只是开发过程敏捷，有些情况下只能做到这个范围，甚至有一些团队这都做不到。需要根据实际情况，下面做的内容是我们的一些探索与实践，站在研发中心角度，我们怎么来拆墙，开门，开窗打孔。

首先是体系框架图。

在信通院DevOps体系规范里面有这样一个体系框架图。我们也基本上延用了相关内容，但是跟它比有一些特色。因为我们企业的形式其它企业不一样，我们是一个研发中心，就是上面有业务部门，下面有运维部门。

例如：

* 在我们的规范中叫组织文化。原版体系架构叫企业文化；
* 工具里面一站式工具平台，按照我们的工具平台写的；
* 再下面是流水线，我们是从提交、一直到部署和运维，而不是更长的一个流程；
* 随后是技术架构、应用架构，我们是有使用自己的Saas，IaaS，PaaS。

通过体系框架的比较，大家可以看到，我们跟业界方式方法基本一样，同时，根据相关实际情况进行了一些改造和优化。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

其次是三条主线，工具、流程、规范。

* 在工具方面，我们要建设统一的平台：DevOps集成平台；
* 在流程方面，建设持续交付流水线、推进自动化测试、完善运营监控；
* 在规范方面，建立一个质量视图和打造DevOps组织规范。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  


**三、三条主线，即：工具、流程、规范**

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  


  


### 3.1 实施路线——工具部分

工具方面：我们会根据规划进行一个逐步收敛，比如：配置管理工具、代码白盒检查、构建和发布工具等。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

同时，

进行管理链和开发链的一体化，测试链与开发链一体化，形成研发侧的工具链。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

之后，就是

研发态和运营态，对应就是研发工具链和运营工具链，形成统一的DevOps平台。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### 3.2 实施路线——流程部分

流程方面：有了工具，还需要用流程进行规范。

比如：持续交付的流程，大家都是差不多，从业务需求，通过编码构建然后一直到测试环境，一直部署到生产部署。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里有一点需要说明，部署到生产环境需要符合银保监会相关要求。

测试方面、运营监控方面处理思路与此类似。

### 3.3 实施路线——规范部分

规范方面：规范部分可能这里面比较重要的就是质量视图，大家都知道-PDCA。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

我们统一的过程管理整体视图是这样的，质量视图是分层级的，例如管理类视图，首先是制作“势能大”、对其它工作有指导作用的领导层使用的报表，通常是与考核和绩效有关的报表和指标。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在开发类视图中，大部分是技术方面的。比如配置管理工具、代码扫描工具、安全扫描工具、构建工具等，它们为项目组、团队来提供运行情况的信息。旁边，有生产环境有运营链对应的报表和指标。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

质量体系中质量视图有指标，大家都关心指标。我们做了大致的分类，预计会形成这样一个视图，横向是我们的指标所产生的阶段，纵向是指标的大分类。比如说周期，会有什么样的指标，跨的范围是什么样的，效率会在哪个方面，不同颜色表示着不同的关注程度。有些指标可以促进系统内部改进和系统之间对比让大家相互促进。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

如果把整个图画出来、把所有指标全都标出来肯定是比较大的图。可以把一些，最后关注点里面的指标先列出来，大家能看到我们关注什么，并且关注是在什么阶段。

这是我们的仪表板样例，包括周期、项目数量、交付的时间、以及所属部门等等。通过度量体系和相关仪表板。我们可以满足领导层进而是各个成绩了解实时情况。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

下面说谈一下

DevOps组织规范，正中间是DevOps的文化建设和体系建设。

文化建设又包含了我们的一些制度标准、敏捷培训体系、指标与监控等方面内容；技术体系方面包括DevOps集成平台、持续交付流水线、分层自动化测试体系、监控运维分析、统一质量视图等。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/tibrg3AoIJTuFebPiawmA9aemwZiaYrJicziam0SiahjWS6oIz8TTdvkKwgY4JTibCAqMn9zp7ic1vY8ricbcZd1ctbViczg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

它们共同作用、在技术方面有DevOps平台，在文化建设方面有标准。还需要定期的外部和内部自检、评价，逐步建立我们DevOps的相关规范。

  


**四、总结**

![](https://mmbiz.qpic.cn/mmbiz_png/3MRbgjUiaA2XictGc0H4D2icx1ZFPS2ia92qIXp9ywTPagWCoKRVvc7TbrGWo7GPxdww6hy8UKt1s0oxBClqaTjydw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


  


输出成果方面，建立农业银行的DevOps成熟度评价体系。

可以评估某个研发系统的DevOps成熟度，这个大家可以对比一下原来的CMMI，我们做这个东西更多是让内部有一个评价、评估和考核方面的需要。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/tibrg3AoIJTuFebPiawmA9aemwZiaYrJicziaElT0jbJ6LdA4odLEKFPOhf73Cy1fma17iaslyicxG8tPribF2enWFwVAA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在考核体系方面，主要考核内容是交付周期，什么时候接到业务需求，一直到什么时候交付给业务。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/tibrg3AoIJTuFebPiawmA9aemwZiaYrJiczia6XrRmq8DGApZe2c4CuqthqUcOiaez46fYfZBsmxgRhIIXd4jDAAvBPw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

今天与大家交流了最开始的企业使用DevOps、敏捷等最重要的“零”；

之后，介绍了1+3，即：

一个体系框架加上工具、流程、规范。

希望能对大家有所帮助。

  


**五、Q & A**

![](https://mmbiz.qpic.cn/mmbiz_png/3MRbgjUiaA2XictGc0H4D2icx1ZFPS2ia92qIXp9ywTPagWCoKRVvc7TbrGWo7GPxdww6hy8UKt1s0oxBClqaTjydw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  


  


**Q1：**

**我们公司也是大型国企，现在领导已经确定了要上DevOps，但因为我们有众多的外包合作方。**

**我们希望借助这个平台和工具链，把很多项目外包模式逐步转成人力外包。**

**在这个中间，有您里面提到的组织文化建设和规范推动的过程，所以我就想慢慢向合作方推或者是要改变很多不同的现有工作模式，有没有什么好的建议？**

A：

领导为什么要推DevOps？解决了企业的一个什么样的痛点？这个痛点是不是真的痛或者是痒点？

大型企业的关注点的可能比较多，我们

要分清，是否真的要解决问题和解决什么问题才去做DevOps。

外包是DevOps需要注意的一个坑，我个人觉得的重要的是，你要怎么来考虑激励他?外包人员如何融入？

这些是跟企业内部员工完全不一样的。

你把这个方面解决一下，人员激励能够融入团队里面来，做到这点才能解决你那个问题，逐渐把供应商的技术能力固化到你们企业里。

激励方面才是最重要的事情，要不然也不要考虑其他的了。

想掌握的产品，跟你的技术能力是要匹配的，我们的特点是大型商业银行，需要承担社会责任、需要遵照银保监会相关要求，我们会一定会满足“风险可控”，同时也会进行“make or buy”分析，采取恰当的方式。

**Q2：**

**您觉得市场上随着DevOps不断发展，是如何改变开发人员和运维人员之间的一种合作方式？**

A：

开发人员和运维人员想改变的就一个，原来大家目标是不一样，开发受业务要求，要尽快发布新的功能，运维那边希望尽可能稳定，别出事，目标不一致导致原来协作方式出现那堵墙。

如何让他们目标一致协作起来，根本原因蛮简单，让这堵墙没有，原来是因为目标不一致产生的墙，现在让目标一致。

怎么一致？就是涉及到企业文化、团队组成、考核，才可能把这墙打破。

如果只是通过工具，刚开始是孔，把信息传递过去，原来信息都传递不过去怎么弄？

现在通过工具孔，把信息传递过去之后，逐步让墙变得透风了，慢慢建窗、门。

慢慢的推倒这堵墙，一定要让大家目标一致。

这个虽然看起来虚，但就是根本。

协调目标这个事情是一个大手笔，通常需要改变组织结构。

**Q3：**

**我比较好奇，在大型金融机构里头，目前在机构里推DevOps都是研发机构发起的，为什么不是数据中心发起呢？**

A：

这个问题真的是一个好问题，Patric发起DevOps时是从一个运维侧逐渐发展起来的。

大家做所有相关实践，大部分都是从运维角度考虑。

这个没错，那么为什么银行这边大部分是从研发这个角度来？

我个人理解不一定对，是因为整个银行压力传导点与前面说的情况不一样。

刚才说DevOps的时候是面临着运维压力，然后说要改，银行这边压力点是在开发部门，并且是业务给开发端的压力。

为什么业务会给开发端这些压力？

原因是多方面的，其中肯定是业务需求旺盛和或业务交付速度不够快。

假如说是这样的情况，对比同业交付速度比较快，而银行业务略微慢一些。

这个时候压力就会出来，交付的压力会传导给谁？

肯定是研发中心。

我们就面临着所有的整个业务，这时，运维方面还没有传到到呢、或者感受的比较小。

