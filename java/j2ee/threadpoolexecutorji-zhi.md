整体处理流程:

① 当一个任务被提交到线程池时，首先查看线程池的核心线程是否都在执行任务，否就选择一条线程执行任务，是就执行第二步。

② 查看核心线程池是否已满 , 不满就创建一条线程执行任务，否则执行第三步。

③ 查看任务队列是否已满，不满就将任务存储在任务队列中，否则执行第四步。

④ 查看线程池是否已满，不满就创建一条线程执行任务，否则就按照策略处理无法执行的任务。

  


在ThreadPoolExecutor中表现为:

① 如果当前运行的线程数小于corePoolSize，那么就创建线程来执行任务（执行时需要获取全局锁）。

② 如果运行的线程大于或等于corePoolSize，那么就把task加入BlockQueue。

③ 如果创建的线程数量大于了BlockQueue的最大容量,  那么创建新线程来执行该任务。

④ 如果创建线程导致当前运行的线程数超过maximumPoolSize，就根据饱和策略来拒绝该任务。

  


创建线程池:

① 首先要提供maxCorePoolSize，maxinumPoolSize，keepAliveTime，BlockQueue这几个线程运行所需的必要参数。

② 可选参数：饱和策略（当任务无法被执行时的处理方式）。

  


① ThreadPoolExecutor.AbortPolicy，用于被拒绝任务的处理程序，它将抛出RejectedExecutionException。

② ThreadPoolExecutor.CallerRunsPolicy用于被拒绝任务的处理程序，它直接在execute方法的调用线程中运行被拒绝的任务。

③ DiscardOldestPolicy用于被拒绝任务的处理程序，它放弃最旧的未处理请求，然后重试 execute。

④ DiscardPolicy用于被拒绝任务的处理程序，默认情况下它将丢弃被拒绝的任务。

  


提交任务:

① 无返回值的任务使用execute（Runnable）

② 有返回值的任务使用submit（Runnable）

  


关闭线程池:

调用shutdown或者shutdownNow，两者都不会接受新的任务，而且通过调用要停止线程的interrupt方法来中断线程，有可能线程永远不会被中断，不同之处在于shutdownNow会首先将线程池的状态设置为STOP，然后尝试停止所有线程（有可能导致部分任务没有执行完）然后返回未执行任务的列表。而shutdown则只是将线程池的状态设置为shutdown，然后中断所有没有执行任务的线程，并将剩余的任务执行完。

  


线程池的配置:

① 如果是CPU密集型任务，那么线程池的线程个数应该尽量少一些，一般为CPU的个数+1条线程。

② 如果是IO密集型任务，那么线程池的线程可以放的很大，如2\*CPU的个数。

③ 混合型任务的话，如果可以拆分的话，可以通过拆分成CPU密集型和IO密集型两种来提高执行效率；如果不能拆分的的话就可以根据实际情况来调整线程池中线程的个数。

  


监控线程池的状态:

① taskCount：线程需要执行的任务个数。

② completedTaskCount：线程池在运行过程中已完成的任务数。

③ largestPoolSize：线程池曾经创建过的最大线程数量。

④ getPoolSize获取当前线程池的线程数量。

⑤ getActiveCount：获取活动的线程的数量

然后通过继承线程池，重写beforeExecute，afterExecute和terminated方法来在线程执行任务前，线程执行任务结束，和线程终结前获取线程的运行情况，根据具体情况调整线程池的线程数量。

