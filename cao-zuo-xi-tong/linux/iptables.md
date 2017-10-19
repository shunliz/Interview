iptables中定义有表，分别表示提供的功能，有filter表（实现包过滤）、nat表（实现网络地址转换）、mangle表（实现包修改）、raw表（实现数据跟踪），这些表具有一定的优先级：raw--&gt;mangle--&gt;nat--&gt;filter

一条链上可定义不同功能的规则，检查数据包时将根据上面的优先级顺序检查


