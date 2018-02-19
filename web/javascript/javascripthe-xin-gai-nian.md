## JavaScript核心概念归纳整理 {#activity-name}

### ![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5EjqiaHGg35UrJrxBUiayzniaHxHjPwjFic4Ultf8L778mRIjjiaiberuQlRQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

JavaScript语言本身是一个庞大而复杂的知识体系，复杂程度不低于任何一门后端语言，本文针对JavaScript语言的核心概念进行简单的梳理，对应的每个知识点仅仅点到为止，不作详细介绍。目的是为了方便大家快速审查自己对JavaScript的知识结构是否完善，如有遗漏或不正确的地方，还望批评指正。

JavaScript核心概念归纳整理

1. **数据类型、判断方法**

2. **执行上下文**

3. **变量对象、活动对象**

4. **原型、原型链**

5. **作用域、作用域链**

6. **闭包、垃圾回收机制**

7. **this指向**

8. **类和模块**

9. **继承**

10. **函数式编程**

11. **同步异步**

12. **JS正则表达式**

13. **事件模型**

14. **Ajax、跨域访问**

15. **DOM**

16. **BOM**

数据类型、判断方法

ECMAScript的基本数据类型有5种：**Undefined、Null、Boolean、Number、String**。

其中Boolean、Number、String属于原始类型，Undefined、Null属于原始值。

原始类型代表了各自类型的所有成员，原始值则代表了各自特殊类型的唯一成员。

ECMAScript的复合数据类型有1种：Object（对象类型）。

Object是一种复合值，它将很多值（原始类型/值或者其他对象）聚合在一起，通过属性的形式进行访问。

ECMAScript的特殊对象类型：**Array、Function、Math、Date、JSON、RegExp、Error**，每种类型都各自代表一种独立的类，不同的类实例拥有不同的类特性以及对应的操作方式。

**ECMAScript常见数据类型划分方式：**

> 1.原始类型、对象类型
>
> 2.值类型、引用类型
>
> 3.可变类型、不可变类型
>
> 4.可拥有方法类型、不可拥有方法类型

**精确区分数据类型的判断方法：Object.prototype.toString.call**

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA51u8rZLaI71zjhx58GgWApKfo6BtMdkpIzqQ43qwEsWu85HmJvFnvCQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

执行上下文

JS的执行上下文可以理解为当前代码的执行环境，在执行JS程序时，每遇到一段JS可执行代码，都会创建一个可执行上下文。JS当中可执行代码分为三种：全局代码、函数代码、eval代码。所以一段JS程序必定会产生多个执行上下文，而JavaScript引擎则是以堆栈的形式来对其进行管理，也就是常说的函数调用栈。栈底是全局上下文，栈顶则是当前正在执行的上下文。例如：

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5o2GuLdYib8bI03RnfKVPkVVG9icmkFnaOjLGZoXKOSkk6jVnPzY3hicEQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

执行上下文在函数调用栈中的顺序为（自底向上）：**globalStack =&gt; threeStack =&gt; twoStack =&gt; oneStack**

**特性**

> 1.单线程
>
> 2.同步执行，只有栈顶的上下文处于执行中，其他上下文需要等待
>
> 3.全局上下文只有唯一的一个，它在浏览器关闭时出栈
>
> 4.函数的执行上下文的个数没有限制
>
> 5.每次某个函数被调用，就会有个新的执行上下文为其创建，即使是调用的自身函数，也是如此。

变量对象、活动对象

在介绍变量对象与活动对象前，首先我们需要更深入的理解执行上下文的生命周期，执行上下文的生命周期分为两个阶段：

第一个阶段是**创建阶段**，每当JS引擎在执行一段可执行代码时，都会先进入创建阶段。该阶段会分别创建变量对象，建立作用域链，以及确定this的指向，作用域链和this指向会在后文阐述。所谓变量对象就是用于存储在执行上下文中定义的变量和函数声明，在当前上下文中每找到一个变量声明，就会在变量对象中建立一个同名的属性，每找到一个函数声明，就会建立一个以函数名命名的属性，属性值则为指向该函数所在内存地址的引用。这些预先建立好的属性以及属性值，存储着该上下文中所有的变量数据，为后续代码的执行奠定基础。

第二个阶段是**执行阶段**，当变量对象，作用域链，this指向都建立之后，执行上下文会进入到执行阶段。在该阶段中变量对象会转化为活动对象，此时活动对象中的属性都允许被访问，并且可以执行其他数据性的操作。

**两者区别：**

执行上下文处于创建阶段时，变量对象中的属性是不允许被访问的，但是在进入到执行阶段后，变量对象转化为活动对象，并且里面的属性都允许被外界访问。其实两者都属于同一个对象，只是处于执行上下文的不同生命周期而已。

原型、原型链

在JavaScript中，每一个对象都会和另一个对象产生关联，从另一个对象上继承属性，这里所指的另一个对象就是我们耳熟能详的**原型**。原型本身也是一个对象，其他对象可以通过它实现属性的继承，也可以将任何一个对象作为自身对象的原型。JS中的任何对象都有原型，除了原型链顶端的对象：Object.prototype

所谓**原型链**，就是由对象原型所构成的访问链，我称之为“原型继承链”。一个JS对象的原型指向其父类对象，而父类对象的原型又指向父类对象的父类对象，这种通过原型层层连接起来的关系就是原型链。

**以下是几种获取原型对象的方法：**

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5VBK7BQwQbBvrupWvzYNtSVSmxoeTxSkfI6bEIrQj7vfcJOWqjrOmEg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

作用域、作用域链

说到**作用域**，就必须结合变量的访问权限来说明。一个变量的作用域就是在程序中定义变量的区域，它规定了执行程序如何对变量进行查找，也就是确定当前的执行代码对变量的访问权限。在ES5中有全局作用域、函数作用域、eval作用域，在ES6中新增了块级作用域。

**作用域链**，则需要结合函数的嵌套来说明。当定义一个函数时，它实际上创建了一个作用域节点，该节点上存储着当前作用域中的局部变量，并且该节点会挂载在作用域链的底端。在该函数中嵌套定义另一个函数时，同样会创建另一个函数作用域的节点，该节点同样也存储着当前函数作用域中的局部变量，在作用域链中会将该节点挂载在外层函数的节点之下。所以在进行变量访问时，会从自身节点开始查找，如果未找到变量的对应值，则会继续查找上一个节点。而由这一系列节点所串联起来的链就是我们所说的作用域链。

JavaScript中的函数采用**静态作用域**，也称词法作用域。当在执行函数调用时，不管何时何地执行函数，其中的变量在函数定义时就已经决定了，函数会从自身作用域节点开始，沿着作用域链向上访问变量的值。

**注意：**作用域链的顶端是全局作用域，作用域链在函数定义时就已经创建了。

闭包、垃圾回收机制

**闭包**，又一个老生常谈的话题，可以用一句话对之概括：有权访问另一个函数作用域内变量的函数都是闭包。例如：

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5BIic2VfxlNia3oHxgWwaXCrPJvkAVBxoLnB1mvnbesT2ibrMTjfyT7ziaQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

这里返回的inner函数就是能够访问outer函数中变量的闭包，除inner函数之外的外部作用域都无法访问outer函数中的变量a。

**闭包特性：**

1.函数返回嵌套的函数形成闭包

2.闭包内部可以访问外部的参数和变量

3.外部参数和变量在被闭包引用时不会被垃圾回收机制回收

**闭包优点：**

1.可避免变量对全局的污染

2.允许函数私有成员的存在

3.允许变量长驻内存

**闭包缺点：**

由于变量常驻内存，增大内存使用量，使用不当很容易造成内存泄漏。

**闭包应用场景：**

1.采用函数引用方式的setTimeout调用

2.将函数关联到对象的实例方法

3.封装相关的功能集

**JS垃圾回收机制原理**

JavaScript中的垃圾回收，主要是一种针对程序执行环境中内存的管理机制，该机制最大限度的优化了JS程序对操作系统内存的使用。垃圾回收机制也同样非常容易理解：就是利用垃圾收集器，周期性的回收那些程序中，不被其他引用所指向的变量的内存资源。不被其他引用所指向的变量就是程序中不会再用到的变量，也就是生命周期结束的变量，这种变量多为局部变量，而全局变量只有在关闭浏览器或终止当前运行环境的情况下其生命周期才会结束。所以此时垃圾收集器所要做的就是周期性的检索程序中处于结束状态的变量，同时回收他们所占用的内存资源。

而闭包的使用则无疑会增加程序对内存资源的占用，因为在闭包中存储着对外部变量的引用，所以只要闭包中存储的外部引用未停止使用，那么外部变量就永远存在，且其所占用的内存资源无法被垃圾回收机制所释放。因此合理的使用闭包，能优化程序的执行效率及降低程序的资源占有率。

this指向

this的指向问题无疑是JavaScript语言中必须掌握的核心概念。上文提到，在执行上下文创建的阶段，就会建立this指向。而更细致的说，this的指向，是在函数被调用的时候确定的。

**下面是this指向的四种场景：**

1.如果一个函数中有this，但是它没有以对象方法的形式调用，而是以函数名的形式执行，那么this指向的就是全局对象。

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5mlwD7VgEU0cibAibA4n3qIVTKfGjk2x049icvWibloGNeyfibRdU9ejeCPA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

2.如果一个函数中有this，并且这个函数是以对象方法的形式调用，那么this指向的就是调用该方法的对象。

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5Cm6iafDUZhDiamrMibjdiaQ8atq4Rwx3VicNyMkG0iaUg6KicHj1dmsv0hiaFg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

3.如果一个函数中有this，并且包含该函数的对象也同时被另一个对象所包含，尽管这个函数是被最外层的对象所调用，this指向的也只是它上一级的对象。

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5CBlzGIs8U7LW03cBF0yCibAlkCUQdhGrKthoNu33DyaIs3xcPorIvpg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

4.如果一个构造函数或类方法中有this，那么它指向由该构造函数或类创建出来的实例对象。

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5o1J5ThLFrqsxCjyFht8viaVK0kdqyPK3uNQOQcTffNFcH2Wzzoibjupw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

类和模块

**类的概念**

**        
**

JavaScript是一种弱类型语言，其本身并不像Java等语言那样对数据具有很强的类型区分，所以为了能够具有面向对象的编码风格，以其独有的方式实现了类的机制。在JavaScript中，类的实现是基于原型（prototype）继承机制的，如果两个实例都从同一个原型对象上继承了属性，可以说它们是属于同一个类的实例。类让每一个成员对象都共享某些属性，这种属性共享的方式在编程中占有举足轻重的地位。

**ES5中的类**：在ES5中，类是由函数来定义的，定义类的函数称之为构造函数。一般这类函数会以首字母大写的形式出现，普通的函数和方法都是以小写字母开头，对象实例化时通过new关键字来调用构造函数。构造函数上挂载着一个prototype属性，该属性存放的是当前类的原型对象，原型对象是类的核心，用于为每一个实例对象提供公有属性。原型对象中还拥有一个constructor属性，用于指向当前类的构造函数。构造函数是类的“公共标识”，而原型对象是类的“唯一标识”。以下是一个类，用于表示点的坐标：

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA55eQEuVmzYn8Vns20EsaXD79En6fdMiaU2K3GxSEuWiaMMOYobhDqqopA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

**ES6中的类**：在ES6中，类的表示就更具语义化，写法上更类似于传统的面向对象语言。它引入了class关键字作为类的标识，并将ES5中prototype的constructor属性直接作为其内部的构造函数，并且在定义类方法时不需要添加function关键字，类方法之间也不需要用逗号进行分隔。类中的静态方法和静态属性用static关键字表示，一旦类函数和类属性用static关键字标记后，实例对象将不会继承这些属性和方法，只能通过类本身来调用。同样用ES6中的类来表示上述的例子：

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5pIG9OCWVZsOGxD62NVOyhdib58DWCvRC9GNhytJkE2WD7p2suwBA6Bg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

**模块化**

**        
**

JavaScript模块化的历史由来已久，也并非小编用几行的篇幅就能一语带过，在这仅对它的特性及应用场景进行笼统的说明，如有对其原委感兴趣的读者，可以搜索其他更详细的相关资料。

先来说**CommonJS**，CommonJS模块化规范主要应用于服务器端编程，加载模块的方式属于同步加载，只有在加载完成之后才能执行后续操作。一个.js文件就是一个CommonJS模块，在服务器端的模块文件一般都保存在本地硬盘，所以加载速度较快。每一个模块都有自己的作用域，里面定义的变量、函数、类都是私有的，对其他文件不可见。NodeJS、webpack就是以CommonJS规范的形式来实现的。

**        
**

**CommonJS模块特点：**

**        
**

> 所有代码都运行在模块作用域，不会污染全局作用域。
>
> 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
>
> 模块加载的顺序，按照其在代码中出现的顺序。

在浏览器环境中，对于模块的下载很大程度上取决于网速的快慢，因此极有可能出现长时间等待现象，从而阻塞浏览器的渲染。所以就必须采用异步模式（AMD、CMD）。AMD模块规范采用异步加载方式，主要用于客户端浏览器环境下，但既可用于浏览器端也可用于服务端，CMD则专注于浏览器端的模块化开发。

**AMD和CMD的区别：**两者的区别在于对模块的加载和执行方式不同，AMD会在加载完模块的同时去执行模块，从而拥有延迟低、效率高的特性；CMD则是加载完所有依赖模块后，再进入程序，遇到需要执行的模块才会执行相应的操作。

**requireJS**是基于AMD规范实现的模块加载器

**seaJS**是基于CMD规范实现的模块加载器

继承

上文说道，由于js本身并不像其他传统的面向对象语言那样，生来就具备类的概念。所以在实现继承的同时，需要用到js的原型及prototype机制或apply、call、bind方法来实现。这里直接上码，介绍几种常见的继承方式：

**类式继承**

**        
**

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5OIibZz9Oe5I4oqNUeVFgQ8OMiaM7YpiaCKCPhl8aBicofejfX2I6UbIx4Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

**原型链继承**

**        
**

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5u4KSDhhmbF1MpTMLZaQ6cT5OsXHOicfXmib313Yf6279F3bmnq4Ov46A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

**继承**

**        
**

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5nkUicSvaOOKghkTVCgHyQjZ2W34VLWRUcRQ8uyl9dX0H7zR6yIPLKAw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

函数式编程

JavaScript并不是专门的函数式编程语言，但却能够应用函数式编程技术，像对象一样去操控函数。下面就例举一些JS函数式编程的典型应用：

**使用非函数式的方式计算数组中元素的平均值和标准差**

**        
**

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5cG8CibbAbHOBe79UxakxjL9cYPN9HbErqma1fVvD3ic0H2iaZmJuoDDLA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

**使用函数式的编程方式**

**        
**

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5xu8RYD4Zxrfdz3scH3dngLcia1DeFJjUy4fRWENTrzRWlib2bcMtlF1w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

同步异步

单线程是JavaScript语言的一大特点，也就是在同一时间只能做一件事。所谓同步，原意就是程序的执行顺序与书写顺寻保持一致；而异步，指的是程序并非按照书写的顺序来执行，会存在“跳过”执行的现象。

谈及JS异步，就不得不提两个改变执行顺序的基础函数：setTimeout和setInterval函数。这两个函数在执行时会被压入事件循环队列当中，在当前作用域下的所有程序都执行完成后，才会开始执行排列在事件循环队列当中的函数，所以这两个函数能够改变程序的执行顺序。来看例子：

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5lXSO5zykASzUSkK9CKr5aoD1CaWVPdldK9F2lRo3Tib9licibPUm5iaDPw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

JS正则表达式

正则表达式大家应该不会陌生，JavaScript中的正则表达式用RegExp类的实例对象来表示，可以使用RegExp\(\)构造函数来创建，也可以用直接量表达式来创建。它的主要功能是用来描述或匹配一段符合某个语法规则的字符串，多用于在一段较长的文本中检索或替换那些符合模式的字符串内容，也经常用于用户的输入校验。例如，使用如下代码来解析一个URL：

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5HxkpII1N4YlfFj48ShKnQsdmykBHdfrAl14I13Piafh6yvDiaGQ8cZ0Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

事件模型

JavaScript的事件模型，是所有前端研发工程师必须必须必须弄清楚的一个基础核心概念！下面为了能让读者迅速回忆起这部分相关的理论知识，笔者将以关键词的形式进行阐述。

**DOM事件流**：当用户触发事件时，该事件首先会从最顶层的document对象开始，自顶向下沿着DOM树的结构逐层传播，直到触发事件的DOM节点对象。之后会从触发事件的DOM节点对象开始，自底向上逐层传播，最后返回到最顶层document对象为止。这就是整个事件流传播的过程。

**事件捕获**：从最顶层的document对象开始，自顶向下沿着DOM树的结构逐层传播，直到触发事件的DOM节点对象的过程。

**事件冒泡**：从触发事件的DOM节点对象开始，自底向上逐层传播，最后到达最顶层的document对象的过程。

**阻止事件流传播**：既可以阻止事件捕获阶段的传播，也可以阻止事件冒泡阶段的传播。在支持\`addEventListener\(\)\`方法的浏览器中，调用事件对象的\`stopPropagation\(\)\`方法阻止事件传播。在IE9之前，设置事件对象的\`cancelBubble\`属性为true来实现阻止事件进一步传播。

**阻止事件的默认行为**：在支持\`addEventListener\(\)\`的浏览器中，调用事件对象的\`preventDefault\(\)\`方法取消事件的默认操作。在IE9之前，设置事件对象的\`returnValue\`属性为false来阻止事件的默认行为。

**事件委托/事件代理**：利用事件冒泡的原理，将事件加到目标节点的父级节点上，触发执行效果。好处就是：（1）可以减少事件绑定的次数，利于提高性能。（2）新添加的元素还会有之前的事件。

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5mPLfjCY5qlAkkLibHxMqCuOwseB1Q2qsAcx9RgTCHh7osdKhDwq1kDA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

Ajax、跨域访问

**Ajax**是浏览器专门用来和服务器进行交互的异步通讯技术，其核心对象是XMLHttpRequest，通过该对象可以创建一个Ajax请求。为了防止XSS攻击，浏览器对Ajax做了限制，不允许Ajax跨域请求服务器,就是只能访问当前域名下的url。

JS的跨域访问，就是在不同的域名下进行HTTP请求与响应。JSONP就是一种常用的跨域通信方式，他利用了脚本跨域能力来模拟Ajax请求。

**JSONP原理**：由于Ajax请求受到同源策略的限制，所以无法跨域访问数据。但能够注意到引入js脚本的\`&lt;script&gt;\`标签拥有跨域请求并执行js文件的能力，所以可以利用这一特点来达到异域间数据交互的目的。\`&lt;script&gt;\`标签中的\`src\`属性就是实现跨域的关键因素。由于字符型的JSON数据能够很好的被原生JS支持，所以服务端只需将数据封装成JSON格式，并塞入到一个回调函数中供客户端请求调用即可。示例如下：

**客户端页面代码如下**

**        
**

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5Bvm0CMO30EQpLhVO9w3ntckAwhOvGvvZhvkO93UttlTr3wib6ztCGlA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

**服务端需要拼接的回调函数及返回数据**

**        
**

![](http://mmbiz.qpic.cn/mmbiz_jpg/BC87ECNQcz0vbWmMJNS6fkoEb9CorAA5KUQNg4nXFCxeSibG0Vp7Tv6rJPm9DL74ibRwtttNWV6iavXVQ7ntEgEQw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

**注意：**这里用callback参数字段将客户端需要执行的回调函数名传给服务端，服务端只需在封装好json数据后，根据该字段值动态创建同名的回调函数即可。

DOM

文档对象模型（document object model）是用来表示和操作HTML和XML文档内容的基础API。Document类型代表了一个HTML或XML文档，document对象则是用来保存整个web页面的dom结构，在页面上所有的元素最终都会映射为一个dom对象。对页面节点的操作也是通过document对象中的方法来实现的。

document对象中常用的Dom操作方法有：getElementById\(\);getElementsByClassName\(\);querySelector\(\);getAttribute\(\);等等。

BOM

浏览器对象模型（browser object model）是用于和浏览器窗口进行交互的对象，也可用于窗口与窗口之间的通信。它的核心对象是window，在window对象当中也提供了很多其他对象属性用于操作和管理浏览器的各个部分。常用的window对象属性有：

**Location对象**：表示该窗口中当前显示的文档的URL.。

**History对象**：用于将窗口的历史浏览记录用文档和文档状态列表的形式表示。

**Navigator对象**：该对象包含了浏览器厂商和版本信息。

**Screen对象**：它提供了有关窗口显示大小和可用的颜色数量信息。

常用的对话框也属于挂载在window对象上的方法：alert\(\); confirm\(\); prompt\(\);

