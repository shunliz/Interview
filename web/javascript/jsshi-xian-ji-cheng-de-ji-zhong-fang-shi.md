前言

JS作为面向对象的弱类型语言，继承也是其非常强大的特性之一。那么如何在JS中实现继承呢？让我们拭目以待。

## JS继承的实现方式

既然要实现继承，那么首先我们得有一个父类，代码如下：

```
// 定义一个动物类
function Animal (name) {
  // 属性
  this.name = name || 'Animal';
  // 实例方法
  this.sleep = function(){
    console.log(this.name + '正在睡觉！');
  }
}
// 原型方法
Animal.prototype.eat = function(food) {
  console.log(this.name + '正在吃：' + food);
};
```

### 1、原型链继承

**核心：**将父类的实例作为子类的原型

```
function Cat(){ 
}
Cat.prototype = new Animal();
Cat.prototype.name = 'cat';

//　Test Code
var cat = new Cat();
console.log(cat.name);
console.log(cat.eat('fish'));
console.log(cat.sleep());
console.log(cat instanceof Animal); //true 
console.log(cat instanceof Cat); //true
```

特点：

1. 非常纯粹的继承关系，实例是子类的实例，也是父类的实例
2. 父类新增原型方法/原型属性，子类都能访问到
3. 简单，易于实现

缺点：

1. 要想为子类新增属性和方法，必须要在
   `new Animal()`
   这样的语句之后执行，不能放到构造器中
2. 无法实现多继承
3. 来自原型对象的引用属性是所有实例共享的（详细请看附录代码：[示例1](javascript:void%280%29;)）
4. 创建子类实例时，无法向父类构造函数传参

推荐指数：★★（3、4两大致命缺陷）

**2017-8-17 10:21:43补充：感谢**[**MMHS**](http://home.cnblogs.com/u/1066372/)** 指出。缺点1中描述有误：可以在Cat构造函数中，为Cat实例增加实例属性。如果要新增原型属性和方法，则必须放在**`new Animal()`**这样的语句之后执行。**

### 2、构造继承

**核心：**使用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类（没用到原型）

```
function Cat(name){
  Animal.call(this);
  this.name = name || 'Tom';
}

// Test Code
var cat = new Cat();
console.log(cat.name);
console.log(cat.sleep());
console.log(cat instanceof Animal); // false
console.log(cat instanceof Cat); // true
```

特点：

1. 解决了1中，子类实例共享父类引用属性的问题
2. 创建子类实例时，可以向父类传递参数
3. 可以实现多继承（call多个父类对象）

缺点：

1. 实例并不是父类的实例，只是子类的实例
2. 只能继承父类的实例属性和方法，不能继承原型属性/方法
3. 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

推荐指数：★★（缺点3）

### 3、实例继承

**核心：**为父类实例添加新特性，作为子类实例返回

```
function Cat(name){
  var instance = new Animal();
  instance.name = name || 'Tom';
  return instance;
}

// Test Code
var cat = new Cat();
console.log(cat.name);
console.log(cat.sleep());
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); // false
```

特点：

1. 不限制调用方式，不管是
   `new 子类()`
   还是
   `子类()`
   ,返回的对象具有相同的效果

缺点：

1. 实例是父类的实例，不是子类的实例
2. 不支持多继承

推荐指数：★★

### 4、拷贝继承

```
function Cat(name){
  var animal = new Animal();
  for(var p in animal){
    Cat.prototype[p] = animal[p];
  }
  Cat.prototype.name = name || 'Tom';
}

// Test Code
var cat = new Cat();
console.log(cat.name);
console.log(cat.sleep());
console.log(cat instanceof Animal); // false
console.log(cat instanceof Cat); // true
```

特点：

1. 支持多继承

缺点：

1. 效率较低，内存占用高（因为要拷贝父类的属性）
2. 无法获取父类不可枚举的方法（不可枚举方法，不能使用for in 访问到）

推荐指数：★（缺点1）

### 5、组合继承

**核心：**通过调用父类构造，继承父类的属性并保留传参的优点，然后通过将父类实例作为子类原型，实现函数复用

```
function Cat(name){
  Animal.call(this);
  this.name = name || 'Tom';
}
Cat.prototype = new Animal();

// 感谢 @学无止境c 的提醒，组合继承也是需要修复构造函数指向的。

Cat.prototype.constructor = Cat;

// Test Code
var cat = new Cat();
console.log(cat.name);
console.log(cat.sleep());
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); // true
```

特点：

1. 弥补了方式2的缺陷，可以继承实例属性/方法，也可以继承原型属性/方法
2. 既是子类的实例，也是父类的实例
3. 不存在引用属性共享问题
4. 可传参
5. 函数可复用

缺点：

1. 调用了两次父类构造函数，生成了两份实例（子类实例将子类原型上的那份屏蔽了）

推荐指数：★★★★（仅仅多消耗了一点内存）

### 6、寄生组合继承

**核心：**通过寄生方式，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化两次实例方法/属性，避免的组合继承的缺点

```
function Cat(name){
  Animal.call(this);
  this.name = name || 'Tom';
}
(function(){
  // 创建一个没有实例方法的类
  var Super = function(){};
  Super.prototype = Animal.prototype;
  //将实例作为子类的原型
  Cat.prototype = new Super();
})();

// Test Code
var cat = new Cat();
console.log(cat.name);
console.log(cat.sleep());
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); //true

感谢 @bluedrink 提醒，该实现没有修复constructor。

Cat.prototype.constructor = Cat; // 需要修复下构造函数
```

特点：

1. 堪称完美

缺点：

1. 实现较为复杂

推荐指数：★★★★（实现复杂，扣掉一颗星）





[JS面向对象编程之：封装、继承、多态](https://www.cnblogs.com/Leo_wl/p/5734794.html)

 最近在实习公司写代码，被隔壁的哥们吐槽说，代码写的没有一点艺术。为了让我的代码多点艺术，我就重新温故了《javascript高级程序设计》（其中几章），然后又看了《javascript设计模式》，然后觉得要写点心得体会，来整理自己所学的吧。以下是我个人见解，错了请轻喷，欢迎指出错误，乐于改正。

![](https://images2015.cnblogs.com/blog/816397/201607/816397-20160731220533138-1790206129.png)



      一、封装

      （1）封装通俗的说，就是我有一些秘密不想让人知道，就通过私有化变量和私有化方法，这样外界就访问不到了。然后如果你有一些很想让大家知道的东西，你就可以通过this创建的属性看作是对象共有属性和对象共有方法，这样别人知道你的公共的东西啦，不止如此，你还可以访问到类或对象自身的私有属性和私有方法。哇，这种权利好大呀，外面的公共的方法和属性，和内部的私有属性和方法都可以访问到，都有特权啦，因此就叫做特权方法了。看个例子就知道啦。

![](https://images2015.cnblogs.com/blog/816397/201607/816397-20160731221941169-1981451310.png)

类的内部this上定义的属性和方法自然就可以复制到新创建的对象上，成为对象公有化的属性和方法，又可以访问私有属性和私有方法，因此就叫特权方法。

这样调用就可以啦

![](https://images2015.cnblogs.com/blog/816397/201607/816397-20160731222522778-478754452.png)

      （2）闭包实现的封装

　　闭包是有权访问另外一个函数作用域中变量的函数，即在一个函数内部创建另外一个函数。这时就可以将闭包作为创建对象的构造函数，这样它既是闭包又是可实例对象的函数。

![](https://images2015.cnblogs.com/blog/816397/201607/816397-20160731230614356-1409630772.png)

       二、继承

　　（1）类

　　 每个类有3个部分：1,是构造函数内的，是供实例化对象复制用的。2,是构造函数外的，直接通过点语法添加的，这是供类使用的，实例化对象是访问不到的。3,是类的原型中的，实例化对象可以通过其原型链简介地访问到，也是为供所有实例化对象所共有的。

     （2）类式继承

     通过子类的原型prototype对象实例化来实现的

![](https://images2015.cnblogs.com/blog/816397/201607/816397-20160731231414903-1734538422.png)

继承就是声明2个类，不过类式继承需要将第一个类的实例赋值给第二个类的原型。这段代码，在实现subClass继承superClass时是通过将superClass的实例赋值给subClass的原型prototype,所以subClass.prototype继承了superClass.

**缺点**就是：一个子类的实例原型从父类构造函数中继承来的共有属性就会直接影响到其他子类。比如：

![](https://images2015.cnblogs.com/blog/816397/201607/816397-20160731232427403-1713790397.png)

**额外知识点**：instanceof是通过对象的prototype链来确定这个对象是否是某个类的实例，而不关心对象与类的自身结构。

       （3）构造函数式继承

       构造函数式继承是通过在子类的构造函数作用环境中执行一次父类的构造函数来实现的。

![](https://images2015.cnblogs.com/blog/816397/201607/816397-20160731233137638-668815399.png)

SuperClass.call\(this,id\);是构造函数式继承的精华，call可以更改函数的作用环境。这个对SuperClass调用这个方法就是将子类中的变量子啊父类中执行一遍，由于父类中是给this绑定属性的，因此子类自然也就继承了父类的共有属性。由于这种类型的继承没有涉及原型prototype,所以父类的原型方法自然不会被子类继承，而如果要想被子类继承就必须要放在构造函数中。

　　（4）组合继承

    组合继承就是：类式继承+构造函数继承

![](https://images2015.cnblogs.com/blog/816397/201607/816397-20160731234213216-1128265252.png)

这里用例子来测试下

![](https://images2015.cnblogs.com/blog/816397/201607/816397-20160731234539106-488093682.png)

 果然子类的实例中更改父类继承下来的引用类型属性如books,根本不会影响到其他实例，并且子类实例化过程中又能将参数传递到父类的构造函数中。

        （5）原型式继承

         原型式继承跟类式继承一样，父类对象book中的值类型的属性被复制，引用类型的属性被共有。

![](https://images2015.cnblogs.com/blog/816397/201608/816397-20160802101645668-406639973.png)

          （6）寄生式继承

          通过在一个函数内的过渡对象实现继承并返回新对象的方式，称之为寄生式继承。

         寄生就像寄生虫一样寄托于某个对象内部生长。就是对原型继承的第二次封装，并且在这第二次封装过程中对继承的对象进行了扩展，这样新创建的对象不仅仅有父类中的属性和方法而且还添加了新的属性和方法。

看下下面的例子吧

![](https://images2015.cnblogs.com/blog/816397/201608/816397-20160802102508778-920595923.png)

             （7）寄生组合式继承

          寄生组合式继承就是寄生式继承+构造函数式继承，

![](https://images2015.cnblogs.com/blog/816397/201608/816397-20160802105736122-504725171.png)

先创建了父类，还有父类的原型方法，然后创建子类，并在构造函数中实现构造函数式继承，然后又通过寄生式继承了父类 原型，最后又对子类添加了一些原型方法。

现在我们来测试一下

![](https://images2015.cnblogs.com/blog/816397/201608/816397-20160802110120465-1702254787.png)

显然不会出现子类调用之后，另一个子类的值被改变。在这里其中最大的改变是对子类原型的处理，被赋予父类原型的一个引用，这是一个对象。

         （8）多继承

![](https://images2015.cnblogs.com/blog/816397/201608/816397-20160802112718090-303191191.png)

          通过这种方式对一个对象属性的复制继承，将多个父类\(对象\)的属性与方法拷贝给子类实现继承

           三、多态

          多态就是通过对传递的参数判断来执行逻辑，即可实现一种多态处理机制

           下面就是这个例子，通过多态类，调用add运算方式，根据不同参数做运算

![](https://images2015.cnblogs.com/blog/816397/201608/816397-20160802113538512-1706925925.png)



这就是面向对象的三种特性啦，封装、继承、多态，对原理的理解，能在看其他人的优秀代码的时候，有个很好的理解。

