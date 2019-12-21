“

最近和朋友聊天，提到他前几天面试的时候被问到：“能否描述一下Spring Cloud？”他当场就懵了，不知道从何说起。

  


![](https://mmbiz.qpic.cn/mmbiz_jpg/MOwlO0INfQrX2f52lpbdpiaYMiaiahuDgh2A9D3D1fjt8Ebt2N9aMR1pfdlKicLTN2icIybKEjwtnoicnya7a7eoxHdQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_图片来自 Unsplash_

  


是啊，Spring Cloud 是知名的微服务架构，包含了很多组件，每个组件又有各自的分工。

  


怎么才能理解 Spring Cloud 架构并且说清楚它到底做了些什么呢？我们今天一起来看一下。

  


从一个例子开始

  


对于这样的“大”问题，通常需要拆解成小问题来回答。要说明 Spring Cloud 做了什么，就要说清楚它包含的组件都做了些什么？

  


如果一个个把组件罗列出来，似乎太过独立，没有关联性，缺少逻辑感。我们就从一个简单的例子开始，把这些组件像串珍珠一样串起来。

  


假设有一个项目，这个项目有两个服务，分别是“A”和“B”：

* **“A”和“B”的关系是，“A”调用“B”。**

* **然后，有一个客户端“C”调用“A”。**

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQ6TRrpH6rIkZHYUzp6ntlZFGV7X1dysBO1xPMFZicgaglNhtKsyfNOnw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_客户端“C”调用服务“A”，服务“A”调用服务“B”_

  


Eureka 服务之间互相认识

  


在服务端我们已经有了两个服务，“A”和“B”。他们的关系是“A”调用“B”，“B”被“A”调用。

  


当只有两个服务的时候我们是知道这种关系，并且可以把这种关系记录下来的，但是如果服务一多，我们如何记录这种关系呢？

  


于是，Eureka 就登场了，它负责“服务注册，服务发现”的工作。Eureka 分成 Eureka Server 和 Eureka Client。

  


每个微服务架构都会有一个或者多个 Eureka Server 用来保存注册服务的信息。

  


每个服务都会包含一个Eureka Client，其中会配置Eureka Server的信息，这样当服务启动的时候就能够把自己注册到 Eureka Server 中去了。

  


同时每个服务也可以通过 Eureka Client 从 Eureka Server 中获取其他服务的信息（Get Registry）。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQ3bX2bBE1CCnwk5qwkRicQsS7hBICg3CuIBsLSEof2yMDb9qkHUP131g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_“A”服务与“B”服务的调用关系_

  


“A”服务和“B”服务首先通过自身集成的 Eureka Client 到 Eureka Server 上注册自身的信息，包括：服务名，地址，端口号等等。

  


注册完毕以后，“A”服务通过 Eureka Client 从 Eureka Server 获取（Get Registry）服务“B”的信息。

  


由于，“A”服务调用“B”服务，所以“A”服务称之为“消费者”，“B”服务称之为“生产者”。

  


Feign 服务之间信息传递

  


既然“A”“B”两个服务互相认识了，接下来就要轮到“A”服务调用“B”服务了。

  


由于两个是单独的服务，并且两个服务都在一个网络内，通常会通过 HTTP 请求进行调用。

  


传统的做法是，“A”服务写好请求的消息，序列化成二进制的串传递给“B”服务，“B”服务收到消息以后反序列化消息进行解析，接着以同样的方式应答“A”服务。

  


从传统意义上完成这些代码需要大量的工作，而且需要考虑很多编码上面的问题。为了简化上面的过程，Feign 组件就诞生了，它方便了服务之间的调用。

  


引入 Feign 了以后，在“A”服务中只需要填写简单的 URL，参数，请求方式，就可以调用“B”服务了。调用“B”服务就好像调用本地的一个方法一样简单。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQ4pHecWs4YwMtZYRYkHFAy6n39DHdU5XtjciaC9SYSxlWeNIrRwP4dyw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_通过 Feign 调用服务的代码片段_

  


我们需要做的就是在“A”服务（消费者）中建立一个 Feign Client，填写我们需要调用的 URL，参数和方式就可以了，其他的事情就交给 Feign 来完成了。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQlFZibuocdxureiaZLmGzgiaglZLtgf144icHM3IVAnlnR4wEcPojH65F4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_Feign Client 工作流_

  


从上图可以看出，“消费者”开始只需要提供“生产者”的 URL，参数等信息。

  


Feign Client 会根据这些信息生成对应的 HTTP 请求头和报文，然后发送给生产者。

  


生产者返回信息以后，Feign Client 同样会返回“消费者”能够读懂的 JavaBean 的信息。

  


Ribbon 搞定负载均衡

  


好了，现在“A”“B”服务互相认识了，并且“A”服务可以调用“B”服务了。

  


假设“B”服务的业务量增大，一个“B”服务无法满足现在的要求，另外又复制了两个“B”服务，连同原来的一个“B”服务，现在一共有三个“B”服务。

  


虽然三个提供的服务都是一样的，但是“A”服务应该调用哪个“B”服务的复制呢？

  


这时 Ribbon 就登场了，它用来做负载均衡。“A”服务无需知道调用三个复制中的哪一个，它只用告诉 Ribbon 我要调用“B”服务，Ribbon 会根据策略去调用三个复制中的某一个。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQGevV6MypvaowXRmpkOtI6jBibKx1pcvIyVnKvB609AvNvNed5ic6SurA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_Ribbon 充当负载均衡器的角色_

  


在 Ribbon 的几种负载均衡策略中，随机策略是用的比较多的。例如：“B1”，“B2”，“B3”分别是“B”服务的三个复制。

  


“A”服务第一次，调用的是“B1”服务，根据随机策略，第二次就访问“B2”服务，第三次访问“B3”服务，第四次又访问“B1”服务，依次类推，循环往复。

  


下面列出其他几个服务仅供参考，篇幅有限不做赘述：

* **随机（Random）**

* **轮询（RoundRobin）**

* **一致性哈希（ConsistentHash）**

* **哈希（Hash）**

* **加权（Weighted）**

#  

Hystrix 服务出现故障

  


现在“A

”

服务知道如何调用“B1

”

，“B2

”

，“B3

”

三个“B

”

服务的复制了。

  


假如“B2”服务出现故障，“A”服务还可以访问它吗？为了避免单个服务的故障影响到其他服务，Hystrix 就应运而生了。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQZXsaVR3orsutoYYtI8Bjibgstk8HF2icRBBFFpWJw0jFibKuiaiaw8yMbPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_调用“B1”服务失败_

  


Hystrix 俗称熔断器（断路器），当服务不可用或者出现故障时，它提供了响应的处理机制。

  


在微服务架构中，每个服务都有可能依赖其他服务，也有可能多个服务同时依赖一个服务，又或者存在服务之间的连环依赖（A 依赖 B，B 依赖 D）。

  


一旦被依赖的服务出现故障，Hystrix 可以通过预设的处理机制，调整服务的响应。例如：返回错误信息，用缓存或者兜底信息替代服务返回信息。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQDjicluicwFa5wq07KTiaPLZKcsSplMiaBp9BXUYGO4bkOruqk3tMaoqViag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

单个服务依赖多个服务，依赖服务中有一个出现问题，对整体产生影响。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQFOluBkrO7NnGQVJRiawOz6NuaVXYqtkPKqNP0sD4rj3oiawhN3fXvySQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_多个服务依赖单个服务，单个服务出现故障，影响多个服务_

  


应用 Hystrix 只需要两步：

  


**第一步，**在“A”服务（消费者）上定义，调用“B”服务（生产者）出现故障时的处理方法。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQAh9eudnhvfuYD6n0OsS7icAUtvr3WhIKiawDzkcvDRzMb8GXahETGtKQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_调用“B”服务故障处理的方法，代码片段_

  


**第二步，**在“A”服务（消费者）调用“B”服务的方法的 Annotation 上面标注调用失败需要执行的“第一步”的这个方法。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQcbsicibMvE2JsQmVtKBHgf1pgkypluYoUplX9ZBNV1qOq22mH0rumqyw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_声明调用失败方法，代码片段_

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQ8zZp8aqiblP7rdmhicGKnCAHJ6qDsRfT50t9nVKDURhJicXEzSVLeBTjw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_在“A”服务中加入 Hystrix_

  


Zuul 如何访问到微服务

  


上面把服务端的事情说的差不多了，如果“C”客户端需要访问“A”服务，系统通过什么方式告诉它哪个服务是“A”呢？

  


实际上这里缺少一个网关，把“C”客户端与“A”服务链接起来的网关。

  


Zuul 就是这个网关，它的责任是过滤和路由。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQudgYbNm58BwhElwJvBuAA0temWuXHcw52mjIwLH5P8EnnWtwiaMOAUw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_Zuul 是链接客户端和服务端的网关_

  


还记得上面提到的 Eureka 服务注册和服务发现吗？Zuul 可以和 Eureka 一起合作，完成服务路由的工作。

  


首先，“A”服务在 Eureka 进行注册，然后“C”客户端向 Zuul 发起请求，访问“A”服务。Zuul 向 Eureka 获取“A”服务的地址，之后访问“A”服务。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQyckw3JTWxkEhQbLib49a4tianZeUURDluThGSPDBokIfjQ5eVKGXw4dQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_Zuul 与 Eureka 协同工作_

  


Zuul 除了可以做路由，还可以做过滤器，针对权限验证，金丝雀测试都可以用到它。这里简单说说 Zuul 内部的运行机制。

  


Zuul 收到 HTTP 请求以后，会通知 Zuul Servlet 处理，与此同时会生成一个 Request Context 用来记录请求的上下文信息，它会一直保持直到路由结束。

  


Zuul Filter Runner 接到 Zuul Servlet 的通知以后，会从 Request Context 中取请求的信息，并且交给 Filter Processer 处理，它会维护一套过滤和路由的规则，根据这些规则将请求发送到目标的服务。

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQpGUHickf10jO3wAj5ufLO99nJOvuRCy4cHDjq15Tf7o7KqcrkZicZnGw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_Zuul 内部工作原理图_

  


Spring Cloud 微服务架构总结

  


说完了上面这些是不是对 Spring Cloud 理解更加深了。让我们来回顾一下，Spring Cloud 是一个微服务架构，它为微服务开发提供了丰富的组件。

  


其中比较重要的五大组件分别是：

* **Eureka：**服务发现，服务注册。

* **Feign：**服务调用请求。

* **Ribbon：**服务之间负载均衡。

* **Hystrix：**熔断器。

* **Zuul：**服务网关。

  


如果，用我们上面 ABC 的例子来记忆就是，A 调用 B（Eureka），A 发送请求（Feign），B 做横向扩展以后，A 通过（Ribbon）找到 B，B 出现问题 A 通过熔断机制（Hystrix）保证服务调用正常，C 通过 Zuul 找到 A。

  


用一张大图来总结一下：

![](https://mmbiz.qpic.cn/mmbiz_png/MOwlO0INfQq0dngqTavbuMNRSssJ5OvQ2Qn9bFDBwhNoeFu2WOOfk42lkITgTgLCTK1829Gn4GfUwApO2ZHosg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

_通过 ABC 理解 Spring Cloud 的五大组件_

  


还可以把整个流程总结一下，

客户端请求→Zuul

→

Eureka 获取服务

→

Feign 通信

→

Ribbon 负载均衡

→

Hystrix 熔断：

* 用户请求会最新发送给 Zuul，Zuul 是用来做 API 网关的。同时它也可以作为过滤器。

* 微服务的注册操作需要通过 Eureka，作为服务发现和注册中心，一方面记录服务的注册以及健康情况，一方面会协同 Zuul 做好服务访问的工作。

* 微服务之间通讯，需要把数据打包发送，接受以后也需要解包读取信息。这里可以使用 Feign 作为服务通讯的组件，配合 Ribbon 完成通信工作。

* Robbin，其负责微服务集群的负载均衡工作。

* 服务出现故障，例如：业务异常，网络异常等等。需要通过断路器 Hystrix 来实现具体的处理操作，比如通知注册中心服务异常，比如对服务进行降级处理。

  这个时候服务注册发现中心会标记服务异常，再有请求过来就不会发送到有异常的服务上去了。

  同时服务发现注册中心也会定期检查服务的状态，一旦服务恢复状态又把其放到访问队列中。



