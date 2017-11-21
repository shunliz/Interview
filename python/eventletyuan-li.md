###  eventlet.green

从epoll的运行机制可以看出, 要使用异步IO, 必须要将相关IO操作改写成non-blocking的方式. 但是我们用`eventlet.spawn()`的函数, 并没有针对epoll做任何改写, 那eventlet是怎么实现 异步IO的呢?

这也是eventlet这个package最凶残的地方, 它自己重写了python标准库中IO相关的操作, 将它们 改写成支持epoll的模式, 放在eventlet.green中.

比如说, `socket.accept()`被改成了这样

```
def accept(self):
    if self.act_non_blocking:
        return self.fd.accept()
    fd = self.fd
    while True:
        res = socket_accept(fd)
        if res is not None:
            client, addr = res
            set_nonblocking(client)
            return type(self)(client), addr
        trampoline(fd, read=True, timeout=self.gettimeout(),
                       timeout_exc=socket.timeout("timed out"))

```

然后在eventlet.spawn\(\)的时候, 通过 一些高阶魔法和"huge hack", 将这些改写过得模块"patch"到spawn出的greenthread上, 从而 实现epoll的IO多路复用, 相当凶残.

### eventlet并发机制分析

前面说了这么多, 这里可以分析一下eventlet的并发机制了.

eventlet的结构如下图所示

```
 _______________________________________
| python process                        |
|   _________________________________   |
|  | python thread                   |  |
|  |   _____   ___________________   |  |
|  |  | hub | | pool              |  |  |
|  |  |_____| |   _____________   |  |  |
|  |          |  | greenthread |  |  |  |
|  |          |  |_____________|  |  |  |
|  |          |   _____________   |  |  |
|  |          |  | greenthread |  |  |  |
|  |          |  |_____________|  |  |  |
|  |          |   _____________   |  |  |
|  |          |  | greenthread |  |  |  |
|  |          |  |_____________|  |  |  |
|  |          |                   |  |  |
|  |          |        ...        |  |  |
|  |          |___________________|  |  |
|  |                                 |  |
|  |_________________________________|  |
|                                       |
|   _________________________________   |
|  | python thread                   |  |
|  |_________________________________|  |
|   _________________________________   |
|  | python thread                   |  |
|  |_________________________________|  |
|                                       |
|                 ...                   |
|_______________________________________|



其中的hub和greenthread分别对应eventlet.hubs.hub和eventlet.greenthread, 本质都是 一个greenlet的实例.

hub中封装前面提到的epoll, epoll的事件循环是由`hub.run()`这个方法里实现. 每当用户调用 eventlet.spawn\(\), 就会在当前python线程的pool里产生一个新的greenthread. 由于greenthread 里的IO相关的python标准库被改写成non-blocking的模式\(参考上面的`socket.accept()`\).

每当greenthread里做IO相关的操作时, 最终都会返回到hub中的epoll循环, 然后根据epoll中的 IO事件, 调用响应的函数. 具体如下面所示.

`greenthread.sleep()`, 实际上也是将CPU控制权交给hub, 然后由hub调度下一个需要运行的 greenthread.

```
   # in eventlet.hubs.poll.Hub

    def wait(self, seconds=None):
        readers = self.listeners[READ]
        writers = self.listeners[WRITE]

        if not readers and not writers:
            if seconds:
                sleep(seconds)
            return
        try:
            presult = self.poll.poll(int(seconds * self.WAIT_MULTIPLIER))
        except select.error, e:
            if get_errno(e) == errno.EINTR:
                return
            raise
        SYSTEM_EXCEPTIONS = self.SYSTEM_EXCEPTIONS

        for fileno, event in presult:
            try:
                if event & READ_MASK:
                    readers.get(fileno, noop).cb(fileno)
                if event & WRITE_MASK:
                    writers.get(fileno, noop).cb(fileno)
                if event & select.POLLNVAL:
                    self.remove_descriptor(fileno)
                    continue
                if event & EXC_MASK:
                    readers.get(fileno, noop).cb(fileno)
                    writers.get(fileno, noop).cb(fileno)
            except SYSTEM_EXCEPTIONS:
                raise
            except:
                self.squelch_exception(fileno, sys.exc_info())
                clear_sys_exc_info()
```



