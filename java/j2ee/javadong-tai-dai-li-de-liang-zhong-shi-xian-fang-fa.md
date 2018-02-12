# Java动态代理的两种实现方法

AOP的拦截功能是由java中的动态代理来实现的。说白了，就是在目标类的基础上增加切面逻辑，生成增强的目标类（该切面逻辑或者在目标类函数执行之前，或者目标类函数执行之后，或者在目标类函数抛出异常时候执行。不同的切入时机对应不同的Interceptor的种类，如BeforeAdviseInterceptor，AfterAdviseInterceptor以及ThrowsAdviseInterceptor等）。

那么动态代理是如何实现将切面逻辑（advise）织入到目标类方法中去的呢？下面我们就来详细介绍并实现AOP中用到的两种动态代理。

AOP的源码中用到了两种动态代理来实现拦截切入功能：jdk动态代理和cglib动态代理。两种方法同时存在，各有优劣。jdk动态代理是由[Java](http://lib.csdn.net/base/java)内部的反射机制来实现的，cglib动态代理底层则是借助asm来实现的。总的来说，反射机制在生成类的过程中比较高效，而asm在生成类之后的相关执行过程中比较高效（可以通过将asm生成的类进行缓存，这样解决asm生成类过程低效问题）。还有一点必须注意：jdk动态代理的应用前提，必须是目标类基于统一的接口。如果没有上述前提，jdk动态代理不能应用。由此可以看出，jdk动态代理有一定的局限性，cglib这种第三方类库实现的动态代理应用更加广泛，且在效率上更有优势。。

# 1、定义接口和实现

```java
package com.meituan.hyt.test3.service;  


public interface UserService {  
    public String getName(int id);  

    public Integer getAge(int id);  
}
```

```java
package com.meituan.hyt.test3.service.impl;  

import com.meituan.hyt.test3.service.UserService;  


public class UserServiceImpl implements UserService {  
    @Override  
    public String getName(int id) {  
        System.out.println("------getName------");  
        return "Tom";  
    }  

    @Override  
    public Integer getAge(int id) {  
        System.out.println("------getAge------");  
        return 10;  
    }  
}
```

# 2、jdk动态代理实现

```java
package com.meituan.hyt.test3.jdk;  

import java.lang.reflect.InvocationHandler;  
import java.lang.reflect.Method;  


public class MyInvocationHandler implements InvocationHandler {  
    private Object target;  

    MyInvocationHandler() {  
        super();  
    }  

    MyInvocationHandler(Object target) {  
        super();  
        this.target = target;  
    }  

    @Override  
    public Object invoke(Object o, Method method, Object[] args) throws Throwable {  
        if("getName".equals(method.getName())){  
            System.out.println("++++++before " + method.getName() + "++++++");  
            Object result = method.invoke(target, args);  
            System.out.println("++++++after " + method.getName() + "++++++");  
            return result;  
        }else{  
            Object result = method.invoke(target, args);  
            return result;  
        }  

    }  
}
```

```java
package com.meituan.hyt.test3.jdk;  

import com.meituan.hyt.test3.service.UserService;  
import com.meituan.hyt.test3.service.impl.UserServiceImpl;  

import java.lang.reflect.InvocationHandler;  
import java.lang.reflect.Proxy;  


public class Main1 {  
    public static void main(String[] args) {  
        UserService userService = new UserServiceImpl();  
        InvocationHandler invocationHandler = new MyInvocationHandler(userService);  
        UserService userServiceProxy = (UserService)Proxy.newProxyInstance(userService.getClass().getClassLoader(),  
                userService.getClass().getInterfaces(), invocationHandler);  
        System.out.println(userServiceProxy.getName(1));  
        System.out.println(userServiceProxy.getAge(1));  
    }  
}
```

运行结果

++++++before getName++++++  
------getName------  
++++++after getName++++++  
Tom  
------getAge------  
10

# 3、cglib动态代理实现

Cglib是一个优秀的动态代理框架，它的底层使用ASM在内存中动态的生成被代理类的子类，使用CGLIB即使代理类没有实现任何接口也可以实现动态代理功能。CGLIB具有简单易用，它的运行速度要远远快于JDK的Proxy动态代理：

CGLIB的核心类：  
    net.sf.cglib.proxy.Enhancer – 主要的增强类
    net.sf.cglib.proxy.MethodInterceptor – 主要的方法拦截类，它是Callback接口的子接口，需要用户实现  
    net.sf.cglib.proxy.MethodProxy – JDK的java.lang.reflect.Method类的代理类，可以方便的实现对源对象方法的调用,如使用：  
    Object o = methodProxy.invokeSuper\(proxy, args\);//虽然第一个参数是被代理对象，也不会出现死循环的问题。

net.sf.cglib.proxy.MethodInterceptor接口是最通用的回调（callback）类型，它经常被基于代理的AOP用来实现拦截（intercept）方法的调用。这个接口只定义了一个方法  
public Object intercept(Object object, java.lang.reflect.Method method,  
Object[] args, MethodProxy proxy) throws Throwable;

第一个参数是代理对像，第二和第三个参数分别是拦截的方法和方法的参数。原来的方法可能通过使用java.lang.reflect.Method对象的一般反射调用，或者使用 net.sf.cglib.proxy.MethodProxy对象调用。net.sf.cglib.proxy.MethodProxy通常被首选使用，因为它更快。

```java
package com.meituan.hyt.test3.cglib;  


import net.sf.cglib.proxy.MethodInterceptor;  
import net.sf.cglib.proxy.MethodProxy;  

import java.lang.reflect.Method;  


public class CglibProxy implements MethodInterceptor {  
    @Override  
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {  
        System.out.println("++++++before " + methodProxy.getSuperName() + "++++++");  
        System.out.println(method.getName());  
        Object o1 = methodProxy.invokeSuper(o, args);  
        System.out.println("++++++before " + methodProxy.getSuperName() + "++++++");  
        return o1;  
    }  
}
```

```java
package com.meituan.hyt.test3.cglib;  

import com.meituan.hyt.test3.service.UserService;  
import com.meituan.hyt.test3.service.impl.UserServiceImpl;  
import net.sf.cglib.proxy.Enhancer;  



public class Main2 {  
    public static void main(String[] args) {  
        CglibProxy cglibProxy = new CglibProxy();  

        Enhancer enhancer = new Enhancer();  
        enhancer.setSuperclass(UserServiceImpl.class);  
        enhancer.setCallback(cglibProxy);  

        UserService o = (UserService)enhancer.create();  
        o.getName(1);  
        o.getAge(1);  
    }  
}
```

运行结果：

++++++before CGLIB$getName$0++++++  
getName  
------getName------  
++++++before CGLIB$getName$0++++++  
++++++before CGLIB$getAge$1++++++  
getAge  
------getAge------  
++++++before CGLIB$getAge$1++++++

# 静态代理

代理模式

代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。

按照代理的创建时期，代理类可以分为两种。

静态代理：由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。

动态代理：在程序运行时，运用反射机制动态创建而成。

```java
package com.meituan.hyt.test3.service;

import com.meituan.hyt.test3.service.impl.UserServiceImpl;

public class UserServiceProxy implements UserService{

    private UserServiceImpl userServiceImpl;  

    @Override
    public String getName(int id) {
        System.out.println("事务处理之前");  
        userServiceImpl.getName(id);
        System.out.println("事务处理之后");  
        return null;
    }

    @Override
    public Integer getAge(int id) {
        System.out.println("事务处理之前");  
        userServiceImpl.getAge(id);
        System.out.println("事务处理之后");  
        return null;
    }

}
```



