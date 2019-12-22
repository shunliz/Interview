Spring Cloud和Kubernetes都是开发和运行微服务的环境，但两者在特性上并不相同，解决的问题点也不一样。本文将探讨这两种平台对于微服务架构的交付有何作用、两者在哪些方面表现更好以及如何利用这两种平台在微服务架构的路上取得成功。

  


预告

8月5日，中国杭州，K8S技术社区联合网易云、EasyStack、DaoCloud，共同打造[**K8S GeekGathering 2017杭州站线下大趴**](http://mp.weixin.qq.com/s?__biz=MzI4NDYxOTgwMw==&mid=2247484121&idx=1&sn=dea12720d66509cf8578c30c0fe976a7&chksm=ebf9e7addc8e6ebb67cba773a0cc687006ff3938a1255e94961cb731196bad5c37c317f27123&scene=21#wechat_redirect)**（点击进入）**，点击“阅读原文”免费报名！

  


**背景故事**

我最近拜读了 A.Lukyanchikov关于如何利用Spring Cloud和Docker搭建微服务的文章，推荐大家也看一看。

想要搭建一个可以十倍、百倍扩展服务的弹性伸缩微服务系统，需要借助具有宽泛构建时间和运行时能力的工具集进行集中的管理和治理。

Spring Cloud包括了各种功能性服务（如统计服务，帐户服务和通知服务）和支持基础设施服务（如日志分析，配置服务器，服务发现，授权服务）。

下图展示使用Spring Cloud的微服务架构：

![](https://mmbiz.qpic.cn/mmbiz_png/Y5pEOlw0RicRyhZt3oGIu6Vh0ypGicPSuTZgnM299wHmCmfJnvmsULnUQSuaq78YOZxwFO9EyAFaWEvkDV5sTYMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

（Spring Cloud微服务架构，by A. Lukyanchikov\)

该图展示了运行时的方方面面，但没有包括打包、持续集成、伸缩、高可用和自我修复等在微服务架构中重要点。本文假设大多数JAVA开发者熟悉Spring Cloud，采用类比的形式，通过解决以上要点问题，带大家了解Kubernetes和Spring Cloud之间的关系。

**微服务要点**

我们在此不进行特性的逐个对比，而是从大面上看一看微服务的要点并聊一聊Spring Cloud和Kubernetes如何实现。

微服务架构的一大优势是易于理解的架构风格，可实现强大的模块边界，并且具有独立的部署和技术多样性，但需要付出的代价也是显而易见的——开发分布式系统的成本和运维开销。

而微服务架构能否成功实践，利用各种工具解决潜在问题是关键。把启动过程变得快速简单很重要，但通往生产环境的旅程是漫长的，你需要不断进步才能成功。

![](https://mmbiz.qpic.cn/mmbiz_png/Y5pEOlw0RicRyhZt3oGIu6Vh0ypGicPSuTHcA8vLlLJuOl5XKtoTmCcHLRsUQubD1FbzN9zTgufzGqagdiccOBd4Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上图是需要在微服务架构中最常见的技术要点（在这里，我们不涉及那些非技术要点，比如组织结构、公司文化等等）

**技术对比**

Spring Cloud和Kubernetes有很大的不同，并没有直接可比的特性，如果对照微服务架构的要点，可以得出如下的技术对比图表：

![](https://mmbiz.qpic.cn/mmbiz_png/Y5pEOlw0RicRyhZt3oGIu6Vh0ypGicPSuT1R3oQqHysew9yv7tmEJukQs95kkdKeQdMHeJhTeFsVUwuRFVvLTOHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从上表我们可以得知：

* Spring Cloud有一套丰富且集成良好的JAVA库，作为应用栈的一部分解决所有运行时问题。因此，微服务本身可以通过库和运行时代理解决客户端服务发现、负载均衡、配置更新、统计跟踪等。工作模式就像单实例服务集群（译者注：集群中master节点工作，当master挂掉后，slave节点被选举顶替。）并且一批工作也是在JVM中被管理。

  

* Kubernetes是多语言的，不仅仅针对Java平台，而是以通用的方式为所有语言解决分布式计算问题。Kubernetes提供了配置管理、服务发现、负载均衡、跟踪、统计、单实例、平台级和应用栈之外的调度工作。该应用不需要任何客户端逻辑的库或代理程序，可以用任何语言编写。

  

* 两个平台依靠相似的第三方工具，如ELK和EFK stacks, tracing libraries等。Hystrix和Spring Boot等库，在两个环境中都表现良好。很多情况下，Spring Cloud和Kubernetes可以形成互补，组建出更强大的解决方案（例如KubeFlix和Spring Cloud Kubernetes）。

* 

**微服务需求**

想要进一步理解Spring Cloud和Kubernetes的适用范围，可以参考下图微服务架构需求。

![](https://mmbiz.qpic.cn/mmbiz_png/Y5pEOlw0RicRyhZt3oGIu6Vh0ypGicPSuT95zXmO4gclzwqNwSibR3GxmXZ4ic0N939SwVJW9Qqru9ltic9v8U6VvibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

有些需求，Spring Cloud表现更好，有需求则是Kubernetes，也有些需求，两者可以用不同的方式满足。好消息是，Spring Cloud和Kubernetes在使用上并不冲突。例如，Spring Cloud提供Maven插件来创建单独JAR应用程序包。结合Docker、Kubernetes的声明式部署和调度能力，轻松运行微服务。同样，Sring Cloud以应用程序内的包装库的形式来支持弹性伸缩，微服务容错使用Hystrix（bulkhead和断路器模式）与Ribbon（负载均衡）。但这些是不够的，当组合Kubernetes健康检查、程序重启和自动伸缩能力，微服务才真正变成一个强壮的系统。

**优缺点**

Spring Cloud

Spring Cloud为开发者提供了快速构建分布式系统中的一些常见模式的工具，例如配置管理，服务发现，断路器，路由等。它是为Java开发人员使用，构建在Netflix OSS库之上的。

**优点**

* Spring Platform提供的统一编程模型和Spring Boot的快速应用程序创建能力，为开发人员提供了很好的微服务开发体验。使用很少的注解，就可以创建一个配置服务器或获得客户端库来配置您的服务。

  

* 丰富的库支持，覆盖大多数运行时需求。Spring Cloud的所有库均由JAVA编写，提供多特性、高控制和易配置。

  

* 不同的Spring Cloud库彼此完全兼容。例如，Feign客户端还将使用Hystrix用于断路器、Ribbon用于负载均衡请求。一切都是注解驱动的，易于Java开发者开发。

**缺点**

* 仅使用JAVA，既是Spring Cloud的优点，也是一大缺陷。微服务架构之所以吸引人，在于按需交换各种技术栈、库，甚至语言的能力。这一点，Spring Cloud做不到。如果你想使用Spring Cloud/Netflix OSS基础设置服务，例如配置管理、服务发现或者负载均衡，解决方案是不优雅的。虽然Netflix Prana项目实现了sidecar模式，显示基于Java客户类库越过HTTP，使得用non-JVM语言编写的应用程序存在于NetflixOSS生态系统变得可能，但它仍然不是很优雅。

  

* 除了JAVA应用程序，还有太多与开发无关的事情需要Java开发人员处理。每个微服务需要运行各种客户端以进行配置检索、服务发现和负载均衡。虽然很容易设置，但这并不会降低对环境的构建时间和运行的依赖性。例如，开发人员可以使用@EnableConfigServer创建一个配置服务器，但这只是开心的假象。每当开发人员想要运行单个微服务时，他们需要启动并运行Config Server。对于受控环境，开发人员必须考虑使Config Server高度可用，并且由于它可以由Git或SVN支持，因此它们需要一个共享文件系统。同样，对于服务发现，开发人员也是需要首先启动Eureka服务器。为了创建一个受控的环境，他们需要在每个AZ上使用多个实例实现集群。可以说，开发人员除了实现所有功能外，还需要额外管理一个复杂的微服务平台。

  

* Spring Cloud目前在微服务方面覆盖的面相对有限，开发人员还需要考虑自动化部署、调度、资源管理、过程隔离、自我修复、构建流水线等，以获得完整的微服务体验。对于这点，我认为拿Spring Cloud和Kubernetes比较是不公平的，应该比较Spring Cloud + Cloud Foundry \(or Docker Swarm\)和Kubernetes。但这也意味着对于一个完整的端到端微服务体验，Spring Cloud必须补充一个像Kubernetes这样的应用程序平台。

Kubernetes

Kubernetes是一个用于自动化部署、扩展和管理容器化应用程序的开源系统。支持多种语言并且提供用于支持、运行、扩展和管理分布式系统的操作系统。

**优点**

* Kubernetes是多语言且语言不敏感的容器管理平台，能够运行云原生和传统的容器化应用程序。Kubernetes提供的服务（如配置管理、服务发现、负载均衡、测试指标收集和日志聚合）可供各种语言使用。这意味着一个平台可以被多个团队（包括使用Spring的Java开发人员）使用，并提供多种用途：应用程序开发、测试环境、构建环境（源码运行、构建服务、依赖仓库）等。

  

* 与Spring Cloud相比，Kubernetes解决了更广的微服务架构问题。除了提供运行时服务，Kubernetes也可以让你制定环境、设置资源限制、RBAC、管理应用程序生命周期、允许自动扩容和自我修复（几乎表现得像一个抗脆弱平台）。

  

* Kubernetes技术基于Google十五年的研发和容器管理经验。此外，Kubernetes有近1000个贡献者，是Github上最活跃的开源社区之一。

**缺点**

* Kubernetes是多语言的，因此它的服务是通用的，并不针对不同的平台（如Spring Cloud for JVM）进行优化。例如，配置会作为环境变量传递给应用程序或挂载的文件系统。它没有像Spring Cloud Config提供的配置更新功能。

  

* Kubernetes不是一个以开发者为中心的平台，更偏向于DevOps的IT人员使用。因此，Java开发人员需要学习一些新的概念，需要学习解决问题的新方法。尽管通过MiniKuber开始一个Kubernetes开发实例很简单，但手动安装一个高可用的Kubernetes集群仍显得有些复杂。

  

* Kubernetes是一个相对较新的平台（2岁），仍然在发展和成长，每个版本都添加了很多新功能，好消息是，这一点已经被考虑到了，Kubernetes的API将是可扩展和向后兼容的。

  

**Spring Cloud和Kubernetes的最佳实践**

如你所见，Spring Cloud和Kubernetes在核心领域都很强，并且正在其他领域努力改进。Spring Cloud可以快速使用，对开发者比较友好；而Kubernetes是DevOps的绝配，虽然学起来可能有点难，但是覆盖了更广泛的微服务技术要点。

![](https://mmbiz.qpic.cn/mmbiz_png/Y5pEOlw0RicRyhZt3oGIu6Vh0ypGicPSuTcopnSmHI63szO4yOfFnwksSDu2FOtN4NKk0azaZc1LY9V0tRM6C4Nw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Spring Cloud和Kubernetes处理了不同范围的微服务架构技术点，而且是用了不同的方法。Spring Cloud方法是试图解决在JVM中的微服务架构要点，而Kubernetes方法是试图让问题消失，为开发者在平台层解决。Spring Cloud在JVM中非常强大，Kubernetes管理那些JVM很强大。看起来各取所长，充分利用这两者的优势是自然而然的趋势了。

![](https://mmbiz.qpic.cn/mmbiz_png/Y5pEOlw0RicRyhZt3oGIu6Vh0ypGicPSuTC2ZbBW3cK9ibsTmRpz6elprYfXB5X1qe1PxhS7eOEhL8pDPKycFdnjQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

结合使用Spring Cloud和Kubernetes，用Spring Cloud提供应用程序打包，Docker和Kubernetes提供部署和调度；Spring通过Hystrix线程池提供应用程序内隔离，Kubernetes通过资源、进程和命名空间隔离；Spring为每个微服务提供健康终端，Kubernetes执行健康检查并且为健康服务的通信提供路由；Spring外部化且升级配置，Kubernetes给每个微服务分配配置……这样的例子还有很多。

