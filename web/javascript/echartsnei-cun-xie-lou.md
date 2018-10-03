# [关于ECharts内存泄漏问题](https://www.cnblogs.com/sysg/p/8608010.html)

　　最近使用websocket加ECharts做了一个实时监控的功能，发现了一个比较严重的问题，就是浏览器运行一段时间就会非常卡，之前在ECharts官网运行官方实例“动态数据 + 时间坐标轴”时，也遇到了同样的情况，只是当时没有当回事，现在来看原来是内存泄漏的问题。那么是什么原因导致的内存泄漏呢？

　　通过上次使用ECharts的经验以及上网查资料得知，原来ECharts在每次setOption后都需要清理变量，在ECharts中是有API手动清理变量的，分别是clear\(\)和dispose\(\)，区别是前者只需插入参数，ECharts就会重绘图表；而后者则是直接将ECharts对象进行清理，需要重新构建ECharts对象。另外，针对IE，也有专门的回收内存函数CollectGarbage，每次浏览器最小化的时候，浏览器都会调用该函数，清理内存。

　　经过亲身试验，综合各种解决方案，挑选了一种个人认为比较好的可行方案，关键部分源码如下：

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
setInterval()或者websocket.onmessage

function
(){
  data0.shift();
  data0.push(data0);
  data1.shift();
  data1.push(data1);
  xdata.shift();
  xdata.push(axisData);
  chart.clear();
//
清空ECharts
  chart.setOption(ismpflowOption);
}

//
IE内存回收机制
if
 (window.CollectGarbage) {  
    setInterval(
function
 () {  
        CollectGarbage();  
    }, 
30000
);  
} 
```

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

　　还有一种方案就是chart.dispose\(\)配合echarts.init\(\),然后再setOption\(\)，也是可行的，与clear\(\)方法的不同之处在于，dispose\(\)方法是销毁ECharts实例，然后再重新初始化，个人觉得clear\(\)方法好一点。

　　以上方法有点问题，内存泄漏的问题确实没有了，但是用户的操作（比如缩放或者点击图例等）在每次执行setInterval\(\)或者websocket.onmessage方法时，ECharts图表都会重置为初始化时的状态，通过查询官方API，找到了解决此问题的方法，代码如下：

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
setInterval()或者websocket.onmessage

function
(){
  xxdata 
= option.xAxis[0
].data;
  data0 
= option.series[0
].data;
  data1 
= option.series[1
].data;
  data0.shift();
  data0.push(newdata0);
  data1.shift();
  data1.push(newdata1);
  xdata.shift();
  xdata.push(newaxisData);
  userOption 
= ismpflow.getOption();
//
返回包含用户操作的option

  userOption.xAxis[0].data =
 xxdata;
  userOption.series[
0].data =
 data0;
  userOption.series[
1].data =
 data1;
  chart.clear();
//
清空ECharts
  chart.setOption(userOption );
}

//
IE内存回收机制
if
 (window.CollectGarbage) {  
    setInterval(
function
 () {  
        CollectGarbage();  
    }, 
30000
);  
} 
```

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

有更好的解决方案欢迎交流！

本文为作者原创，转载请注明出处。

前端交流群，群文件提供大量文档、书籍和资料。期待你的加入！群号：127768464

