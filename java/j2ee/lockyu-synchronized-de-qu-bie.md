# Lock与synchronized 的区别

---

1、ReentrantLock 拥有Synchronized相同的并发性和内存语义，此外还多了 锁投票，定时锁等候和中断锁等候  
线程A和B都要获取对象O的锁定，假设A获取了对象O锁，B将等待A释放对O的锁定，  
如果使用 synchronized ，如果A不释放，B将一直等下去，不能被中断  
如果 使用ReentrantLock，如果A不释放，可以使B在等待了足够长的时间以后，中断等待，而干别的事情  
  
ReentrantLock获取锁定与三种方式：  
a\) lock\(\), 如果获取了锁立即返回，如果别的线程持有锁，当前线程则一直处于休眠状态，直到获取锁  
b\) tryLock\(\), 如果获取了锁立即返回true，如果别的线程正持有锁，立即返回false；  
c\)tryLock\(long timeout,TimeUnit unit\)， 如果获取了锁定立即返回true，如果别的线程正持有锁，会等待参数给定的时间，在等待的过程中，如果获取了锁定，就返回true，如果等待超时，返回false；  
d\) lockInterruptibly:如果获取了锁定立即返回，如果没有获取锁定，当前线程处于休眠状态，直到或者锁定，或者当前线程被别的线程中断  
  
2、synchronized是在JVM层面上实现的，不但可以通过一些监控工具监控synchronized的锁定，而且在代码执行时出现异常，JVM会自动释放锁定，但是使用Lock则不行，lock是通过代码实现的，要保证锁定一定会被释放，就必须将unLock\(\)放到finally{}中  
  
3、在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态；

5.0的多线程任务包对于同步的性能方面有了很大的改进，在原有synchronized关键字的基础上，又增加了ReentrantLock，以及各种Atomic类。了解其性能的优劣程度，有助与我们在特定的情形下做出正确的选择。

总体的结论先摆出来：

synchronized：  
在资源竞争不是很激烈的情况下，偶尔会有同步的情形下，synchronized是很合适的。原因在于，编译程序通常会尽可能的进行优化synchronize，另外可读性非常好，不管用没用过5.0多线程包的程序员都能理解。

ReentrantLock:  
ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock确还能维持常态。

Atomic:  
和上面的类似，不激烈情况下，性能比synchronized略逊，而激烈的时候，也能维持常态。激烈的时候，Atomic的性能会优于ReentrantLock一倍左右。但是其有一个缺点，就是只能同步一个值，一段代码中只能出现一个Atomic的变量，多于一个同步无效。因为他不能在多个Atomic之间同步。



```java
package test.thread;
 
import static java.lang.System.out;
 
import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.locks.ReentrantLock;
 
public class TestSyncMethods {
     
    public static void test(int round,int threadNum,CyclicBarrier cyclicBarrier){
        new SyncTest("Sync",round,threadNum,cyclicBarrier).testTime();
        new LockTest("Lock",round,threadNum,cyclicBarrier).testTime();
        new AtomicTest("Atom",round,threadNum,cyclicBarrier).testTime();
    }
 
    public static void main(String args[]){
         
        for(int i=0;i<5;i++){
            int round=100000*(i+1);
            int threadNum=5*(i+1);
            CyclicBarrier cb=new CyclicBarrier(threadNum*2+1);
            out.println("==========================");
            out.println("round:"+round+" thread:"+threadNum);
            test(round,threadNum,cb);
             
        }
    }
}
 
class SyncTest extends TestTemplate{
    public SyncTest(String _id,int _round,int _threadNum,CyclicBarrier _cb){
        super( _id, _round, _threadNum, _cb);
    }
    @Override
    /**
     * synchronized关键字不在方法签名里面，所以不涉及重载问题
     */
    synchronized long  getValue() {
        return super.countValue;
    }
    @Override
    synchronized void  sumValue() {
        super.countValue+=preInit[index++%round];
    }
}
 
 
class LockTest extends TestTemplate{
    ReentrantLock lock=new ReentrantLock();
    public LockTest(String _id,int _round,int _threadNum,CyclicBarrier _cb){
        super( _id, _round, _threadNum, _cb);
    }
    /**
     * synchronized关键字不在方法签名里面，所以不涉及重载问题
     */
    @Override
    long getValue() {
        try{
            lock.lock();
            return super.countValue;
        }finally{
            lock.unlock();
        }
    }
    @Override
    void sumValue() {
        try{
            lock.lock();
            super.countValue+=preInit[index++%round];
        }finally{
            lock.unlock();
        }
    }
}
 
 
class AtomicTest extends TestTemplate{
    public AtomicTest(String _id,int _round,int _threadNum,CyclicBarrier _cb){
        super( _id, _round, _threadNum, _cb);
    }
    @Override
    /**
     * synchronized关键字不在方法签名里面，所以不涉及重载问题
     */
    long  getValue() {
        return super.countValueAtmoic.get();
    }
    @Override
    void  sumValue() {
        super.countValueAtmoic.addAndGet(super.preInit[indexAtomic.get()%round]);
    }
}
abstract class TestTemplate{
    private String id;
    protected int round;
    private int threadNum;
    protected long countValue;
    protected AtomicLong countValueAtmoic=new AtomicLong(0);
    protected int[] preInit;
    protected int index;
    protected AtomicInteger indexAtomic=new AtomicInteger(0);
    Random r=new Random(47);
    //任务栅栏，同批任务，先到达wait的任务挂起，一直等到全部任务到达制定的wait地点后，才能全部唤醒，继续执行
    private CyclicBarrier cb;
    public TestTemplate(String _id,int _round,int _threadNum,CyclicBarrier _cb){
        this.id=_id;
        this.round=_round;
        this.threadNum=_threadNum;
        cb=_cb;
        preInit=new int[round];
        for(int i=0;i<preInit.length;i++){
            preInit[i]=r.nextInt(100);
        }
    }
     
    abstract void sumValue();
    /*
     * 对long的操作是非原子的，原子操作只针对32位
     * long是64位，底层操作的时候分2个32位读写，因此不是线程安全
     */
    abstract long getValue();
 
    public void testTime(){
        ExecutorService se=Executors.newCachedThreadPool();
        long start=System.nanoTime();
        //同时开启2*ThreadNum个数的读写线程
        for(int i=0;i<threadNum;i++){
            se.execute(new Runnable(){
                public void run() {
                    for(int i=0;i<round;i++){
                        sumValue();
                    }
 
                    //每个线程执行完同步方法后就等待
                    try {
                        cb.await();
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    } catch (BrokenBarrierException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
 
 
                }
            });
            se.execute(new Runnable(){
                public void run() {
 
                    getValue();
                    try {
                        //每个线程执行完同步方法后就等待
                        cb.await();
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    } catch (BrokenBarrierException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
 
                }
            });
        }
         
        try {
            //当前统计线程也wait,所以CyclicBarrier的初始值是threadNum*2+1
            cb.await();
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        //所有线程执行完成之后，才会跑到这一步
        long duration=System.nanoTime()-start;
        out.println(id+" = "+duration);
         
    }
 
}
```



