# 面试

## java多态
封装，继承，多态是java的三大特性；  

Java实现多态有三个必要条件：继承、重写、向上转型。  

多态的实现：继承和借口；  
继承是实现多态的方式之一；

## NIO
总的来说java 中的IO 和NIO的区别主要有3点：  

IO是面向流的，NIO是面向缓冲的；  
IO是阻塞的，NIO是非阻塞的；  
IO是单线程的，NIO 是通过选择器来模拟多线程的；

一般来说 I/O 模型可以分为：同步阻塞，同步非阻塞，异步阻塞，异步非阻塞 四种IO模型。

### 同步阻塞 IO ：

在此种方式下，用户进程在发起一个 IO 操作以后，必须等待 IO 操作的完成，只有当真正完成了 IO 操作以后，用户进程才能运行。 JAVA传统的 IO 模型属于此种方式！

### 同步非阻塞 IO:

在此种方式下，用户进程发起一个 IO 操作以后可以返回做其它事情，但是用户进程需要时不时的询问 IO 操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的 CPU 资源浪费。其中目前 JAVA 的 NIO 就属于同步非阻塞 IO 。

### 异步阻塞 IO ：

此种方式下是指应用发起一个 IO 操作以后，不等待内核 IO 操作的完成，等内核完成 IO 操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问 IO 是否完成，那么为什么说是阻塞的呢？因为此时是通过 select 系统调用来完成的，而 select 函数本身的实现方式是阻塞的，而采用 select 函数有个好处就是它可以同时监听多个文件句柄，从而提高系统的并发性！

### 异步非阻塞 IO:

在此种模式下，用户进程只需要发起一个 IO 操作然后立即返回，等 IO 操作真正的完成以后，应用程序会得到 IO 操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的 IO 读写操作，因为 真正的 IO读取或者写入操作已经由 内核完成了。目前 Java 中还没有支持此种 IO 模型。

java传统的io是基于同步阻塞的io模型；  

## 锁

### 锁的分类
公平锁/非公平锁  
可重入锁  
独享锁/共享锁  
互斥锁/读写锁  
乐观锁/悲观锁  
分段锁  
偏向锁/轻量级锁/重量级锁  
自旋锁  

IO密集型应用适合多线程，因为io操作大多会阻塞线程，cpu利用率低，可以多开线程；
Runnable比Thread优势是java不能多继承，所以线程类不能再继承其他类；
synchronized 不是公平锁，多线程尝试获取，谁抢到是谁的；

暂停线程 suspend  
放弃cpu资源 yield方法  
线程具有优先级，可以通过方法设置线程优先级，cpu会将资源尽量给优先级高的线程，但是当优先级差别不大的时候，优先级高的不一定先执行完run方法  
线程有两种，一种用户线程，一种守护线程，直到用户线程都销毁，守护线程才销毁  
消息同步  
wait和notify必须是在同步方法和同步代码块里面调用，要不然会抛出异常  
notify方法是继承自Object类，可以唤醒在此对象监视器等待的线程，也就是说唤醒的是同一个锁的线程  
notify方法调用之后，不会马上释放锁，而是运行完该同步方法或者是运行完该同步代码块的代码  
调用notify后随机唤醒的是一个线程  
调用wait方法后会将锁释放  
wait状态下中断线程会抛出异常  
wait(long),超过设置的时间后会自动唤醒，还没超过该时间也可以通过其他线程唤醒  
notifyAll可以唤醒同一锁的所有线程  
如果线程还没有处于等待状态，其他线程进行唤醒，那么不会起作用，此时会打乱程序的正常逻辑  
ThreadLocal  

### ReentrantLock  重入锁  
```
public class MyService {

    private Lock lock = new ReentrantLock();
    private Condition condition=lock.newCondition();
    public void testMethod() {
        
        try {
            lock.lock();
            System.out.println("开始wait");
            condition.await();
            for (int i = 0; i < 5; i++) {
                System.out.println("ThreadName=" + Thread.currentThread().getName()
                        + (" " + (i + 1)));
            }
        } catch (InterruptedException e) {
            // TODO 自动生成的 catch 块
            e.printStackTrace();
        }
        finally
        {
            lock.unlock();
        }
    }
}
```
通过创建Condition对象来使线程wait，必须先执行lock.lock方法获得锁  

```
public void signal()
    {
        try
        {
            lock.lock();
            condition.signal();
        }
        finally
        {
            lock.unlock();
        }
    }
```
condition对象的signal方法可以唤醒wait线程  

```
Lock lock=new ReentrantLock(true);//公平锁
Lock lock=new ReentrantLock(false);//非公平锁
```
公平锁指的是线程获取锁的顺序是按照加锁顺序来的，而非公平锁指的是抢锁机制，先lock的线程不一定先获得锁。  


Lock类有读锁和写锁，读读共享，写写互斥，读写互斥  

### synchronized
synchronized关键字是通过字节码指令来实现的  
synchronized关键字编译后会在同步块前后形成monitorenter和monitorexit两个字节码指令  

### volatile  
volatile boolean status = false;  
一个轻量级的同步锁机制；  
volatile禁止重排序；  

保证变量对所有的线程的可见性，当一个线程修改了这个变量的值，其他线程可以立即知道这个新值（之所以有可见性的问题，是因为java的内存模型）

所有变量都存在主内存，每条线程有自己的工作内存，工作内存保存了被该线程使用的变量的主内存副本拷贝  
线程对变量的所有操作都必须在工作内存中进行，不能直接读写主内存的变量，也就是必须先通过工作内存  
一个线程不能访问另一个线程的工作内存  

### Atomic  
AtomicBoolean , AtomicInteger, AtomicLong, AtomicReference  
当有多个线程同时对单个（包括基本类型及引用类型）变量进行操作时，具有排他性，即当多个线程同时对该变量的值进行更新时，仅有一个线程能成功，而未成功的线程可以像自旋锁一样，继续尝试，一直等到执行成功。  
其原理：CAS操作（compare and swap 对比和设置），是通过一个cpu指令实现的，这个指令是一个原子指令，指令有3个操作数ABC，A为内存位置，B为预期值，C为新值，那么用V更新A的值，如果A符合旧预期值B，如果不符合就不更新，这个过程是原子操作  
所以我们并没有通过代码来实现同步，而是通过硬件级别的cpu指令来实现的，并不像synchronized一样阻塞线程  


http://www.cnblogs.com/-new/p/7358255.html

### 线程池  
新任务创建线程与否逻辑  
核心线程池是否已满--> 队列是否已满--> 线程池是否已满  
ThreadPoolExecutor  

配置线程池：  
cpu密集型和io密集型线程数的选择，cpu密集型不需要太多的线程，可以充分利用cpu资源，io密集型适当多线程，io阻塞时可以切换至另一线程。  
优先级不同的的任务可以使用PriorityBlockingQueue来处理  
建议使用有界队列，能够增加系统的稳定性（如果使用无界队列，当出现问题时候，队列无限增长，此时可能会占用大量内存，导致系统出现问题）和预警能力（当出现队列占满的时候，抛出异常，可以让开发人员及时发现）  

CompletionService 可以实现在单线程中将下游任务多线程话  
