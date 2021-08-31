# 并发篇

## 1. 什么是进程、线程、协程，他们之间的关系是怎样的？

- **进程**: 本质上是一个独立执行的程序，**进程是操作系统资源分配的基本单位**。
- **线程**：**线程是任务调度和执行的基本单位**。。它被包含在进程之中，是进程中的实际运作单位。**一个进程中可以并发多个线程**，每条线程执行不同的任务，切换受系统控制。
- 协程：**又称为微线程，是一种用户态的轻量级线程**。协程不像线程和进程需要进行系统内核上的上下文切换，协程的上下文切换是由用户自己决定的，有自己的上下文，所以说是轻量级的线程，也称之为用户级别的线程就叫协程，**一个线程可以多个协程**，线程进程都是同步机制，而协程则是异步 。子程序切换不是线程切换，而是由程序自身控制。

## 2. 并发和并行

- 并发 `concurrency`：**一核CPU，模拟出来多条线程，快速交替执行**。
- 并行 `parallellism`：**多核CPU ，多个线程可以同时执行**；

## 3. java实现多线程的几种方式，区别，哪个常用

### 1. 继承Thread

- 继承 *Thread*，重写里面`run()`方法，创建实例，执行`start()`方法。
- 优点：代码编写最简单直接操作。
- 缺点：**没返回值，继承一个类后，没法继承其他的类**，拓展性差。

```java
public class ThreadDemo1 extends Thread {
    @Override
    public void run() {
        System.out.println("继承Thread实现多线程，名称："+Thread.currentThread().getName());
    }
}

public static void main(String[] args) {
      ThreadDemo1 threadDemo1 = new ThreadDemo1();
      threadDemo1.setName("demo1");
      // 执行start
      threadDemo1.start();
      System.out.println("主线程名称："+Thread.currentThread().getName());
}
```

### 2. 实现Runnable接口

- 自定义类实现 Runnable，实现里面run()方法，创建 Thread 类，使用 Runnable 接口的实现对象作为参数传递给Thread 对象，调用strat()方法。
- 优点：线程类可以实现多个几接口，可以再继承一个类。
- 缺点：没返回值，不能直接启动，需要通过构造一个 Thread 实例传递进去启动。

```java
public class ThreadDemo2 implements Runnable {
    @Override
    public void run() {
        System.out.println("通过Runnable实现多线程，名称："+Thread.currentThread().getName());
    }
}

public static void main(String[] args) {
        ThreadDemo2 threadDemo2 = new ThreadDemo2();
        Thread thread = new Thread(threadDemo2);
        thread.setName("demo2");
    	// start线程执行
        thread.start();
        System.out.println("主线程名称："+Thread.currentThread().getName());
}

// JDK8之后采用lambda表达式
public static void main(String[] args) {
    Thread thread = new Thread(() -> {
        System.out.println("通过Runnable实现多线程，名称："+Thread.currentThread().getName());
    });
    thread.setName("demo2");
    // start线程执行
    thread.start();
    System.out.println("主线程名称："+Thread.currentThread().getName());
}
```

### 3. 实现Callable接口

- 创建 *Callable* 接口的实现类，并实现`call()`方法，结合 **FutureTask** 类包装 *Callable* 对象，实现多线程。
- 优点：**有返回值，拓展性也高**
- 缺点：Jdk5以后才支持，需要重写`call()`方法，结合多个类比如 **FutureTask** 和 *Thread* 类

```java
public class MyTask implements Callable<Object> {
    @Override
    public Object call() throws Exception {
        System.out.println("通过Callable实现多线程，名称："+Thread.currentThread().getName());
        return "这是返回值";
    }
}

public static void main(String[] args) {
    	// JDK1.8 lambda表达式
        FutureTask<Object> futureTask = new FutureTask<>(() -> {
          System.out.println("通过Callable实现多线程，名称：" +
                        			Thread.currentThread().getName());
            return "这是返回值";
        });

     	// MyTask myTask = new MyTask();
		// FutureTask<Object> futureTask = new FutureTask<>(myTask);
        // FutureTask继承了Runnable，可以放在Thread中启动执行
        Thread thread = new Thread(futureTask);
        thread.setName("demo3");
    	// start线程执行
        thread.start();
        System.out.println("主线程名称:"+Thread.currentThread().getName());
        try {
            // 获取返回值
            System.out.println(futureTask.get());
        } catch (InterruptedException e) {
            // 阻塞等待中被中断，则抛出
            e.printStackTrace();
        } catch (ExecutionException e) {
            // 执行过程发送异常被抛出
            e.printStackTrace();
        }
}
```

### 4. 通过线程池创建线程

- 自定义 *Runnable* 接口，实现 `run()`方法，创建线程池，调用执行方法并传入对象。
- 优点：安全高性能，复用线程。
- 缺点： Jdk5后才支持，需要结合 *Runnable* 进行使用。

```java
public class ThreadDemo4 implements Runnable {
    @Override
    public void run() {
        System.out.println("通过线程池+runnable实现多线程，名称：" +
                           Thread.currentThread().getName());
    }
}

public static void main(String[] args) {
    	// 创建线程池
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        for(int i=0;i<10;i++){
            // 线程池执行线程任务
            executorService.execute(new ThreadDemo4());
        }
        System.out.println("主线程名称:"+Thread.currentThread().getName());
        // 关闭线程池
        executorService.shutdown();
}
```

2 和4常用

### Runnable Callable Thread区别

- Thread 是一个抽象类，只能被继承，而 Runable、Callable 是接口，需要实现接口中的方法。
- 继承 Thread 重写run()方法，实现Runable接口需要实现run()方法，而Callable是需要实现call()方法。
- Thread 和 Runable 没有返回值，Callable 有返回值。
- 实现 Runable 接口的类不能直接调用start()方法，需要 new 一个 Thread 并发该实现类放入 Thread，再通过新建的 Thread 实例来调用start()方法。
- 实现 Callable 接口的类需要借助 FutureTask (将该实现类放入其中)，再将 FutureTask 实例放入 Thread，再通过新建的 Thread 实例来调用start()方法。获取返回值只需要借助 FutureTask 实例调用get()方法即

## 4. 线程的几个状态

- 新建状态（New）：线程刚被创建，但尚未启动。如：Thread t = new Thread();
- 就绪状态（Runnable）：当调用线程对象的start()方法后，线程即进入就绪状态。处于就绪状态的线程，只是说明此线程已经做好了准备，随时等待CPU调度执行，并不是说执行了t.start()此线程立即就会执行。
- 运行状态（Running）：当 CPU 开始调度处于就绪状态的线程时，此时线程才得以真正执行，即进入到运行状态。
- 阻塞状态（Blocked）：处于运行状态中的线程由于某种原因，暂时放弃对CPU的使用权，停止执行，此时进入阻塞状态，直到其进入到就绪状态，才有机会再次被CPU调用以进入到运行状态。根据阻塞产生的原因不同，阻塞状态又可以分为三种：
  - 等待阻塞 ：运行的线程执行wait()方法，该线程会释放占用的所有资源，JVM会把该线程放入“等待池”中。进入这个状态后，是不能自动唤醒的，必须依靠其他线程调用notify()或notifyAll()方法才能被唤 醒，wait()是 Object 类的方法。
  - 同步阻塞：当线程处于运行状态时，试图获得某个对象的同步锁时，如果该对象的同步锁已经被其他线程占用，Java虚拟机就会把这个线程放到这个对象的锁池中。
  - 其他阻塞状态：当前线程执行了sleep()方法，或者调用了其他线程的join()方法，或者发出了 I/O 请求时，就会进入这个状态。线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者 I/O处理完毕时，线程重新转入就绪状态。
- 死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。
  

## 5. 线程状态转换方法：sleep/yield/join wait/notify/notifyAll的区别

### Thread下的方法：

- sleep()：属于线程 *Thread* 的方法，让线程暂缓执行，等待预计时间之后再恢复，交出CPU使用权，不会释放锁，抱着锁睡觉！进入超时等待状态TIME_WAITGING，睡眠结束变为就绪Runnable。不会考虑线程的优先级，因此会给低优先级的线程运行的机会
- yield()：属于线程Thread的方法，暂停当前线程的对象，去执行其他线程，交出CPU使用权，不会释放锁，和`sleep()`类似，和 sleep 不同的是 yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。还有一点和 sleep 不同的是 yield 方法只能使同优先级或更高优先级的线程有执行的机会
- join()：属于线程 *Thread* 的方法，在主线程上运行调用该方法，会让主线程休眠，不会释放锁，让调用`join()`方法的线程先执行完毕，再执行其他线程。类似让救护车警车优先通过！！

### Object下的方法：

- wait()：属于 Object 的方法，当前线程调用对象的 wait()方法，会释放锁，进入线程的等待队列，需要依靠notify()或者notifyAll()唤醒，或者wait(timeout)时间自动唤醒。
- notify()：属于 Object 的方法，唤醒在对象监视器上等待的单个线程，随机唤醒。
- notifyAll()：属于Object 的方法，唤醒在对象监视器上等待的全部线程，全部唤醒。

## 6. Thread调用start()方法和run()方法的区别

`run()`：普通的方法调用`run()`函数，在**主线程中执行，不会新建一个线程来执行**。

`start()`：**新启动一个线程，这时此线程处于就绪（可运行）状态，并没有真正运行**，一旦得到 CPU 时间片，就调用 `run()` 方法执行线程任务。



## 7. 线程池

### 线程池的好处

重用存在的线程，减少对象创建销毁的开销，有效的控制最大并发线程数，**提高系统资源的使用率**，同时避免过多资源竞争，避免堵塞，且可以定时定期执行、单线程、并发数控制，配置任务过多任务后的拒绝策略等功能。

### 类别

- newFixedThreadPool ：一个定长线程池，可控制线程最大并发数。
- newCachedThreadPool：一个可缓存线程池。
- newSingleThreadExecutor：一个单线程化的线程池，用唯一的工作线程来执行任务。
- newScheduledThreadPool：一个定长线程池，支持定时/周期性任务执行。

### 【阿里巴巴编码规范】 线程池不允许使用 Executors 去创建，要通过 ThreadPoolExecutor的方式原因？

Executors创建的线程池底层也是调用 ThreadPoolExecutor，只不过使用不同的参数、队列、拒绝策略等如果使用不当，会造成资源耗尽问题。直接使用ThreadPoolExecutor让使用者更加清楚线程池允许规则，常见参数的使用，避免风险。

常见的线程池问题：
1.newFixedThreadPool和newSingleThreadExecutor: 
	队列使用LinkedBlockingQueue，队列长度为 Integer.MAX_VALUE，可能造成堆积，导致OOM
2.newScheduledThreadPool和newCachedThreadPool:
    线程池里面允许最大的线程数是Integer.MAX_VALUE，可能会创建过多线程，导致OOM

### ThreadPoolExecutor中的参数及作用

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

- `corePoolSize`：**核心线程数**，线程池也会维护线程的最少数量，默认情况下核心线程会一直存活，即使没有任务也不会受存 *keepAliveTime* 控制！
  **坑**：在刚创建线程池时线程不会立即启动，到有任务提交时才开始创建线程并逐步线程数目达到 *corePoolSize*。
- `maximumPoolSize`：**线程池维护线程的最大数量**，超过将被阻塞！
  **坑**：当核心线程满，且阻塞队列也满时，才会判断当前线程数是否小于最大线程数，才决定是否创建新线程
- `keepAliveTime`：**非核心线程的闲置超时时间**，超过这个时间就会被回收，直到线程数量等于 *corePoolSize*。
- `unit`：指定 *keepAliveTime* 的单位，如 *TimeUnit.SECONDS、TimeUnit.MILLISECONDS*
- `workQueue`：**线程池中的任务队列**，常用的是 *ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue*。
- `threadFactory`：**创建新线程时使用的工厂**
- `handler`：*RejectedExecutionHandler* 是一个接口且只有一个方法，线程池中的数量大于 *maximumPoolSize*，对拒绝任务的处理策略，

### 拒绝策略

- `AbortPolicy`：中止策略。默认的拒绝策略，直接抛出 *RejectedExecutionException*。调用者可以捕获这个异常，然后根据需求编写自己的处理代码。
- `DiscardPolicy`：抛弃策略。什么都不做，直接抛弃被拒绝的任务。
- `DiscardOldestPolicy`：抛弃最老策略。抛弃阻塞队列中最老的任务，相当于就是队列中下一个将要被执行的任务，然后重新提交被拒绝的任务。如果阻塞队列是一个优先队列，那么“抛弃最旧的”策略将导致抛弃优先级最高的任务，因此最好不要将该策略和优先级队列放在一起使用。
- CallerRunsPolicy：调用者运行策略。在调用者线程中执行该任务。该策略实现了一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将任务回退到调用者（调用线程池执行任务的主线程），由于执行任务需要一定时间，因此主线程至少在一段时间内不能提交任务，从而使得线程池有时间来处理完正在执行的任务。

### 线程池运行流程

![img](https://img-blog.csdnimg.cn/20200608092639652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3YxMjM0MTE3Mzk=,size_16,color_FFFFFF,t_70)

1. 提交任务之后，判断是否达到了线程池核心线程数：否=>创建工作线程，是=>下一步
2. 判断阻塞队列是否已满，否=>加入阻塞队列，是=>下一步
3. 判断是否达到了最大线程数，否=>创建工作线程，是=>执行拒绝策略

## 8. java中保证线程安全的方法

- 加锁：synchronize/ReentrantLock
- volatile声明变量，轻量级同步，不能保证原子性。
- 使用线程安全类
- 使用线程安全集合容器
- ThreadLocal本地私有变量/信号量Semaphore等

## 9. JMM(Java Memory Model)

JMM屏蔽了各种硬件和操作系统访问的差异性，保证了Java程序在各种平台下对内存的访问都能达到一致的效果.

### 硬件内存模型

- 在现代计算机的硬件体系中，CPU的运算速度是非常快的，远远高于它从存储介质读取数据的速度。所以，在程序运行的过程中，CPU大部分时间都浪费在了磁盘IO、网络通讯、数据库访问上，为把CPU的运算能力压榨出来，让CPU同时去处理多项任务则是最容易想到的，就是“并发”。
- 让CPU并发地执行多项任务并不是那么容易实现的事，因为所有的运算都不可能只依靠CPU的计算就能完成，往往还需要跟内存进行交互，如读取运算数据、存储运算结果等。CPU与内存的交互往往是很慢的，所以这就要求我们要想办法在CPU和内存之间建立一种连接，使它们达到一种平衡，让运算能快速地进行，而这种连接就是我们常说的“高速缓存”。
- 高速缓存的速度是非常接近CPU的，但是它的引入又带来了新的问题，现代的CPU往往是有多个核心的，每个核心都有自己的缓存，而多个核心之间是不存在时间片的竞争的，它们可以并行地执行，那么，怎么保证这些缓存与主内存中的数据的一致性就成为了一个难题。为了解决缓存一致性的问题，多个核心在访问缓存时要遵循一些协议，在读写操作时根据协议来操作，这些协议有MSI、MESI、MOSI等，它们定义了何时应该访问缓存中的数据、何时应该让缓存失效、何时应该访问主内存中的数据等基本原则。
- 随着CPU能力的不断提升，一层缓存就无法满足要求了，就逐渐衍生出了多级缓存。这三种缓存的技术难度和制作成本是相对递减的，容量也是相对递增的。当CPU要读取一个数据的时候，先从一级缓存中查找，如果没找到再从二级缓存中查找，如果没找到再从三级缓存中查找，如果没找到再从主内存中查找，然后再把找到的数据依次加载到多级缓存中，下次再使用相关的数据直接从缓存中查找即可。
- 而加载到缓存中的数据也不是说用到哪个就加载哪个，而是加载内存中连续的数据，一般来说是加载连续的64个字节，因此，如果访问一个 long 类型的数组时，当数组中的一个值被加载到缓存中时，另外 7 个元素也会被加载到缓存中，这就是“缓存行”的概念。
- 缓存行虽然能极大地提高程序运行的效率，但是在多线程对共享变量的访问过程中又带来了新的问题，也就是非常著名的“伪共享”(？)。
- 除此之外，为了使CPU中的运算单元能够充分地被利用，CPU可能会对输入的代码进行乱序执行优化，然后在计算之后再将乱序执行的结果进行重组，保证该结果与顺序执行的结果一致，但并不保证程序中各个语句计算的先后顺序与代码的输入顺序一致，因此，如果一个计算任务依赖于另一个计算任务的结果，那么其顺序性并不能靠代码的先后顺序来保证。
- 为了解决上面提到的多个缓存读写一致性以及乱序排序优化的问题，这就有了内存模型，它定义了共享内存系统中多线程读写操作行为的规范。

![图片](https://mmbiz.qpic.cn/mmbiz_png/C91PV9BDK3xbDJsntH0PEMI2Yl8RfVK8TOic3SLeAT6YSCDD0L7s1HEL02SnsuUCzNd3zX6HTLCajYg8EiathAkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### JMM

- Java内存模型定义了程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出这样的底层细节。这里所说的变量包括实例字段、静态字段，但不包括局部变量和方法参数，因为它们是线程私有的，它们不会被共享，自然不存在竞争问题。

- Java内存模型规定了所有的变量都存储在主内存中，这里的主内存跟介绍硬件时所用的名字一样，两者可以类比，但此处仅指虚拟机中内存的一部分。
- 除了主内存，每条线程还有自己的工作内存。工作内存中保存着该线程使用到的变量的主内存副本的拷贝，线程对变量的操作都必须在工作内存中进行，包括读取和赋值等，而不能直接读写主内存中的变量，不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递必须通过主内存来完成。
- 这里所说的主内存、工作内存跟Java虚拟机内存区域划分中的堆、栈是不同层次的内存划分，如果两者一定要勉强对应起来，主内存主要对应于堆中对象的实例部分，而工作内存主要对应与虚拟机栈中的部分区域。
- 从更低层次来说，主内存主要对应于硬件内存部分，工作内存主要对应于CPU的高速缓存和寄存器部分，但也不是绝对的，主内存也可能存在于高速缓存和寄存器中，工作内存也可能存在于硬件内存中。

![图片](https://mmbiz.qpic.cn/mmbiz_png/C91PV9BDK3xbDJsntH0PEMI2Yl8RfVK8Gs8jZ6wXUBoOMQKUiac4WvjryngZRClicpU6KdS2aXiaUNqDpn4WasIaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 内存间的交互操作

（1）lock，锁定，作用于主内存的变量，它把主内存中的变量标识为一条线程独占状态；

（2）unlock，解锁，作用于主内存的变量，它把锁定的变量释放出来，释放出来的变量才可以被其它线程锁定；

（3）read，读取，作用于主内存的变量，它把一个变量从主内存传输到工作内存中，以便后续的load操作使用；

（4）load，载入，作用于工作内存的变量，它把read操作从主内存得到的变量放入工作内存的变量副本中；

（5）use，使用，作用于工作内存的变量，它把工作内存中的一个变量传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作；

（6）assign，赋值，作用于工作内存的变量，它把一个从执行引擎接收到的变量赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时使用这个操作；

（7）store，存储，作用于工作内存的变量，它把工作内存中一个变量的值传递到主内存中，以便后续的write操作使用；

（8）write，写入，作用于主内存的变量，它把store操作从工作内存得到的变量的值放入到主内存的变量中；

如果要把一个变量从主内存复制到工作内存，那就要按顺序地执行read和load操作，同样地，如果要把一个变量从工作内存同步回主内存，就要按顺序地执行store和write操作。

### 原子性、可见性、有序性

Java内存模型就是为了解决多线程环境下共享变量的一致性问题。一致性主要包含三大特性：原子性、可见性、有序性

（1）原子性

原子性是指一段操作一旦开始就会一直运行到底，中间不会被其它线程打断，这段操作可以是一个操作，也可以是多个操作。

由Java内存模型来直接保证的原子性操作包括read、load、user、assign、store、write这两个操作，我们可以大致认为基本类型变量的读写是具备原子性的。如果应用需要一个更大范围的原子性，Java内存模型还提供了lock和unlock这两个操作来满足这种需求，尽管不能直接使用这两个操作，但我们可以使用它们更具体的实现synchronized来实现。

因此，synchronized块之间的操作也是原子性的。

（2）可见性

可见性是指当一个线程修改了共享变量的值，其它线程能立即感知到这种变化。

Java内存模型是通过在变更修改后同步回主内存，在变量读取前从主内存刷新变量值来实现的，它是依赖主内存的，无论是普通变量还是volatile变量都是如此。普通变量与volatile变量的主要区别是是否会在修改之后立即同步回主内存，以及是否在每次读取前立即从主内存刷新。

除了volatile之外，还有两个关键字也可以保证可见性，它们是synchronized和final。

synchronized的可见性是由“对一个变量执行unlock操作之前，必须先把此变量同步回主内存中，即执行store和write操作”这条规则获取的。

final的可见性是指被final修饰的字段在构造器中一旦被初始化完成，那么其它线程中就能看见这个final字段了。

（3）有序性

Java程序中天然的有序性可以总结为一句话：如果在本线程中观察，所有的操作都是有序的；如果在另一个线程中观察，所有的操作都是无序的。

前半句是指线程内表现为串行的语义，后半句是指“指令重排序”现象和“工作内存和主内存同步延迟”现象。

Java中提供了volatile和synchronized两个关键字来保证有序性。

volatile天然就具有有序性，因为其禁止重排序。

synchronized的有序性是由“一个变量同一时刻只允许一条线程对其进行lock操作”这条规则获取的。

### 先行发生原则（Happens-Before）

如果Java内存模型的有序性都只依靠volatile和synchronized来完成，那么有一些操作就会变得很啰嗦，但是我们在编写Java并发代码时并没有感受到，这是因为Java语言天然定义了一个“先行发生”原则，这个原则非常重要，依靠这个原则我们可以很容易地判断在并发环境下两个操作是否可能存在竞争冲突问题。

先行发生，是指操作A先行发生于操作B，那么操作A产生的影响能够被操作B感知到，这种影响包括修改了共享内存中变量的值、发送了消息、调用了方法等。

（1）程序次序原则

在一个线程内，按照程序书写的顺序执行，书写在前面的操作先行发生于书写在后面的操作，准确地讲是控制流顺序而不是代码顺序，因为要考虑分支、循环等情况。

（2）监视器锁定原则

一个unlock操作先行发生于后面对同一个锁的lock操作。

（3）volatile原则

对一个volatile变量的写操作先行发生于后面对该变量的读操作。

（4）线程启动原则

对线程的start()操作先行发生于线程内的任何操作。

（5）线程终止原则

线程中的所有操作先行发生于检测到线程终止，可以通过Thread.join()、Thread.isAlive()的返回值检测线程是否已经终止。

（6）线程中断原则

对线程的interrupt()的调用先行发生于线程的代码中检测到中断事件的发生，可以通过Thread.interrupted()方法检测是否发生中断。

（7）对象终结原则

一个对象的初始化完成（构造方法执行结束）先行发生于它的finalize()方法的开始。

（8）传递性原则

如果操作A先行发生于操作B，操作B先行发生于操作C，那么操作A先行发生于操作C。

这里说的“先行发生”与“时间上的先发生”没有必然的关系。

[参考地址]: https://mp.weixin.qq.com/s?__biz=MzkxNDEyOTI0OQ==&amp;mid=2247484428&amp;idx=1&amp;sn=efceba0b2b7e0b78ba50de3ad69945f5&amp;chksm=c1726c02f605e51488dea05f94c314338f7dbc7f14d3b3b6203e7f4cc8257bc86f39f62ad937&amp;scene=178&amp;cur_album_id=1538024362992254978#rd

## 10.  volatile

volatile的可见性可以通过下面的示例体现：

### 可见性

```java
public class VolatileTest {    
// public static int finished = 0;    
    public static volatile int finished = 0;
    private static void checkFinished() {        
        while (finished == 0) {            
            // do nothing        
        }        
        System.out.println("finished");    
    }
    
    private static void finish() {
        finished = 1;    
    }
    
    public static void main(String[] args) throws InterruptedException {
        // 起一个线程检测是否结束        
        new Thread(() -> checkFinished()).start();
        Thread.sleep(100);
        // 主线程将finished标志置为1        
        finish();
        System.out.println("main finished");
    }
}
```

在上面的代码中，针对finished变量，使用volatile修饰时这个程序可以正常结束，不使用volatile修饰时这个程序永远不会结束。

因为不使用volatile修饰时，checkFinished()所在的线程每次都是读取的它自己工作内存中的变量的值，这个值一直为0，所以一直都不会跳出while循环。

使用volatile修饰时，checkFinished()所在的线程每次都是从主内存中加载最新的值，当finished被主线程修改为1的时候，它会立即感知到，进而会跳出while循环。

### 禁止重排序

```java
public class VolatileTest3 {    
    private static Config config = null;    
    private static volatile boolean initialized = false;
    public static void main(String[] args) {        
        // 线程1负责初始化配置信息        
        new Thread(() -> {            
            config = new Config();           
            config.name = "config";           
            initialized = true;  
        }).start();
  
        // 线程2检测到配置初始化完成后使用配置信息       
        new Thread(() -> {           
            while (!initialized) {         
                LockSupport.parkNanos(TimeUnit.MILLISECONDS.toNanos(100));         
            }
            // do sth with config         
            String name = config.name;     
        }).start();    
    }
}
class Config {    String name;}
```

这个例子很简单，线程1负责初始化配置，线程2检测到配置初始化完毕，使用配置来干一些事。

在这个例子中，如果initialized不使用volatile来修饰，可能就会出现重排序，比如在初始化配置之前把initialized的值设置为了true，这样线程2读取到这个值为true了，就去使用配置了，这时候可能就会出现错误。

（此处这个例子只是用于说明重排序，实际运行时很难出现。）

### 内存屏障

上面讲了volatile可以保证可见性和禁止重排序，那么它是怎么实现的呢？

答案就是，内存屏障。

内存屏障有两个作用：

（1）阻止屏障两侧的指令重排序；

（2）强制把写缓冲区/高速缓存中的数据回写到主内存，让缓存中相应的数据失效；



volatile变量的影响范围不仅仅只包含它自己，它会对其上下的变量值的读写都有影响。

### 缺陷

不能保证原子性

volatile关键字只能保证可见性和有序性，不能保证原子性，要解决原子性的问题，还是只能通过加锁或使用原子类的方式解决。

进而，我们得出volatile关键字使用的场景：

（1）运算的结果并不依赖于变量的当前值，或者能够确保只有单一的线程修改变量的值；

（2）变量不需要与其他状态变量共同参与不变约束。

说白了，就是volatile本身不保证原子性，那就要增加其它的约束条件来使其所在的场景本身就是原子的。

## 11. synchronize

synchronized关键字是Java里面最基本的同步手段，它经过编译之后，会在同步块的前后分别生成 monitorenter 和 monitorexit 字节码指令，这两个字节码指令都需要一个引用类型的参数来指明要锁定和解锁的对象。

### 原理

在学习Java内存模型的时候，我们介绍过两个指令：lock 和 unlock。

lock，锁定，作用于主内存的变量，它把主内存中的变量标识为一条线程独占状态。

unlock，解锁，作用于主内存的变量，它把锁定的变量释放出来，释放出来的变量才可以被其它线程锁定。

但是这两个指令并没有直接提供给用户使用，而是提供了两个更高层次的指令 monitorenter 和 monitorexit 来隐式地使用 lock 和 unlock 指令。

根据JVM规范的要求，在执行monitorenter指令的时候，首先要去尝试获取对象的锁，如果这个对象没有被锁定，或者当前线程已经拥有了这个对象的锁，就把锁的计数器加1，相应地，在执行monitorexit的时候会把计数器减1，当计数器减小为0时，锁就释放了。

### 原子性、可见性、有序性

synchronized关键字底层是通过monitorenter和monitorexit实现的，而这两个指令又是通过lock和unlock来实现的。

而lock和unlock在Java内存模型中是必须满足下面四条规则的：

（1）一个变量同一时刻只允许一条线程对其进行lock操作，但lock操作可以被同一个线程执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才能被解锁。

（2）如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作初始化变量的值；

（3）如果一个变量没有被lock操作锁定，则不允许对其执行unlock操作，也不允许unlock一个其它线程锁定的变量；

（4）对一个变量执行unlock操作之前，必须先把此变量同步回主内存中，即执行store和write操作；

通过规则（1），我们知道对于lock和unlock之间的代码，同一时刻只允许一个线程访问，所以，synchronized是具有原子性的。

通过规则（1）（2）和（4），我们知道每次lock和unlock时都会从主内存加载变量或把变量刷新回主内存，而lock和unlock之间的变量（这里是指锁定的变量）是不会被其它线程修改的，所以，synchronized是具有可见性的。

通过规则（1）和（3），我们知道所有对变量的加锁都要排队进行，且其它线程不允许解锁当前线程锁定的对象，所以，synchronized是具有有序性的。

综上所述，synchronized是可以保证原子性、可见性和有序性的。

## 12. Java里面有哪些锁，介绍下

### 乐观锁/悲观锁

- 悲观锁：
  - 当线程去操作数据的时候，总认为别的线程会去修改数据，所以它每次拿数据的时候总会上锁，别的线程去拿数据的时候就会阻塞，比如synchronized

- 乐观锁：
  - 每次去拿数据的时候都认为别人不会修改，更新的时候会判断是别人是否回去更新数据，通过版本来判断，如果数据被修改了就拒绝更新，比如CAS是乐观锁，但严格来说并不是锁，通过原子性来保证数据的同步，比如说数据库的乐观锁，通过版本控制来实现，CAS不会保证线程同步，乐观的认为在数据更新期间没有其他线程影响
- 小结：悲观锁适合写操作多的场景，乐观锁适合读操作多的场景，乐观锁的吞吐量会比悲观锁大！

### 公平锁/非公平锁

- 公平锁：
  指多个线程按照申请锁的顺序来获取锁，简单来说 如果一个线程组里，能保证每个线程都能拿到锁 比如ReentrantLock(底层是同步队列FIFO: First Input First Output来实现)
- 非公平锁：
  获取锁的方式是随机获取的，保证不了每个线程都能拿到锁，也就是存在有线程饿死，一直拿不到锁，比如synchronized、ReentrantLock
- 小结：非公平锁性能高于公平锁，更能重复利用CPU的时间。ReentrantLock中可以通过构造方法指定是否为公平锁，默认为非公平锁！synchronized无法指定为公平锁，一直都是非公平锁。

### 可重入锁/不可重入锁

- 可重入锁：
  也叫递归锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁。一个线程获取锁之后再尝试获取锁时会自动获取锁，可重入锁的优点是避免死锁。
- 不可重入锁：
  若当前线程执行某个方法已经获取了该锁，那么在方法中尝试再次获取锁时，就会获取不到被阻塞
- 小结：可重入锁能一定程度的避免死锁 synchronized、ReentrantLock都是可重入锁！
  

### 独占锁/共享锁

- 独享锁，是指锁一次只能被一个线程持有。

  也叫X锁/排它锁/写锁/独享锁：该锁每一次只能被一个线程所持有，加锁后任何线程试图再次加锁的线程会被阻塞，直到当前线程解锁。例子：如果 线程A 对 data1 加上排他锁后，则其他线程不能再对 data1 加任何类型的锁，获得独享锁的线程即能读数据又能修改数据！

- 共享锁，是指锁一次可以被多个线程持有。

  也叫S锁/读锁，能查看数据，但无法修改和删除数据的一种锁，加锁后其它用户可以并发读取、查询数据，但不能修改，增加，删除数据，该锁可被多个线程所持有，用于资源数据共享！

**ReentrantLock和synchronized都是独享锁，ReadWriteLock的读锁是共享锁，写锁是独享锁**。

### 互斥锁/读写锁

与独享锁/共享锁的概念差不多，是独享锁/共享锁的具体实现。

**ReentrantLock和synchronized都是互斥锁，ReadWriteLock是读写锁**

### 自旋锁

- 自旋锁：
  一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环，任何时刻最多只能有一个执行单元获得锁。
  不会发生线程状态的切换，一直处于用户态，减少了线程上下文切换的消耗，缺点是循环会消耗CPU。
- 常见的自旋锁：TicketLock，CLHLock，MSCLock

## 13. CAS？ ABA问题

定义：

- CAS操作包含三个操作数————内存位置（V）、期望值（A）和新值（B）。
- 如果内存位置的值与期望值匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不作任何操作。
- 无论哪种情况，它都会在CAS指令之前返回该位置的值。（CAS在一些特殊情况下仅返回CAS是否成功，而不提取当前值）
- CAS有效的说明了 “我认为位置V应该包含值A；如果包含该值，则将B放到这个位置；否则，不要更改该位置的值，只告诉我这个位置现在的值即可。”



### ABA

CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，在CAS方法执行之前，被其它线程修改为了B、然后又修改回了A，那么CAS方法执行检查的时候会发现它的值没有发生变化，但是实际却变化了。这就是CAS的ABA问题。

### 如何解决ABA

解决ABA最简单的方案就是给值加一个修改版本号，每次值变化，都会修改它的版本号，CAS操作时都去对比此版本号。

java中ABA解决方法（`AtomicStampedReference`），这种方式类似于乐观锁，即：**通过当前版本号来控制CAS交换，如果当前版本号与期望版本号相等，才能交换，否则不可以交换，每执行一次交换当前版本号就+1**。

## 14. ReentrantLock和synchronized差别

- ReentrantLock和synchronized都是独占锁，可重入锁，悲观锁
- synchronized：
  1、java内置关键字
  2、无法判断是否获取锁的状态，只能是非公平锁！
  3、加锁解锁的过程是隐式的，用户不用手动操作，优点是操作简单但显得不够灵活
  4、一般并发场景使用足够、可以放在被递归执行的方法上，且不用担心线程最后能否正确释放锁
- ReentrantLock：
  1、是个Lock接口的实现类
  2、可以判断是否获取到锁，可以为公平锁也可以是非公平锁(默认)
  3、需要手动加锁和解锁，且解锁的操作尽量要放在finally代码块中，保证线程正确释放锁
  5、创建的时候通过传进参数true 创建公平锁，如果传入的是false或没传参数则创建的是非公平锁
  6、底层是AQS的 state 和 FIFO 队列来控制加锁。

## 15. 什么是线程的上下文切换

多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。


## 16. 守护线程和用户线程的区别

- 守护线程：运行在后台，为其他前台线程服务。一旦所有用户线程都结束运行，守护线程会随之一起结束运行。

- 用户线程：运行在前台，用于执行具体的任务，如程序的主线程

  可以通过thread.setDaemon(true)方式将一个线程设置为守护线程。

注意①：必须在thread.start()之前设置，否则会跑出一个 IllegalThreadStateException异常。不能把正在运行的常规线程设置为守护线程。

注意②：由于守护线程的终止是自身无法控制的，因此千万不要把 IO、File 等重要操作逻辑分配给它；因为这些操作会随时可能抛出异常，守护线程也会随之结束！


## 18. ThreadLocal

**`ThreadLocal`类主要解决的就是让每个线程绑定自己的值，可以将`ThreadLocal`类形象的比喻成存放数据的盒子，盒子中可以存储每个线程的私有数据。**

**如果你创建了一个`ThreadLocal`变量，那么访问这个变量的每个线程都会有这个变量的本地副本，这也是`ThreadLocal`变量名的由来。他们可以使用 `get（）` 和 `set（）` 方法来获取默认值或将其值更改为当前线程所存的副本的值，从而避免了线程安全问题。**

**最终的变量是放在了当前线程的 `ThreadLocalMap` 中，并不是存在 `ThreadLocal` 上，`ThreadLocal` 可以理解为只是`ThreadLocalMap`的封装，传递了变量值。** `ThrealLocal` 类中可以通过`Thread.currentThread()`获取到当前线程对象后，直接通过`getMap(Thread t)`可以访问到该线程的`ThreadLocalMap`对象。

**每个`Thread`中都具备一个`ThreadLocalMap`，而`ThreadLocalMap`可以存储以`ThreadLocal`为 key ，Object 对象为 value 的键值对。**

### 内存泄漏问题

`ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用,而 value 是强引用。所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，`ThreadLocalMap` 中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。ThreadLocalMap 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法



## 19. Atomic原子类

在我们这里 Atomic 是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

### JUC包中的原子类是哪四类

**基本类型**

使用原子的方式更新基本类型

- `AtomicInteger`：整形原子类
- `AtomicLong`：长整型原子类
- `AtomicBoolean`：布尔型原子类

**数组类型**

使用原子的方式更新数组里的某个元素

- `AtomicIntegerArray`：整形数组原子类
- `AtomicLongArray`：长整形数组原子类
- `AtomicReferenceArray`：引用类型数组原子类

**引用类型**

- `AtomicReference`：引用类型原子类
- `AtomicStampedReference`：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
- `AtomicMarkableReference` ：原子更新带有标记位的引用类型

**对象的属性修改类型**

- `AtomicIntegerFieldUpdater`：原子更新整形字段的更新器
- `AtomicLongFieldUpdater`：原子更新长整形字段的更新器
- `AtomicReferenceFieldUpdater`：原子更新引用类型字段的更新器

### AtomicInteger类的原理

AtomicInteger 类主要利用 CAS (compare and swap) + volatile 和 native 方法来保证原子操作，从而避免 synchronized 的高开销，执行效率大为提升。

## 20. AQS

AQS的全称是AbstractQueuedSynchronizer，它的定位是为Java中几乎所有的锁和同步器提供一个基础框架。

AQS是基于FIFO的队列实现的，并且内部维护了一个状态变量state，通过原子更新这个状态变量state即可以实现加锁解锁操作。

### 原理

**AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

![AQS原理图](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/AQS%E5%8E%9F%E7%90%86%E5%9B%BE.png)

AQS 使用一个 int 成员变量来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 CAS 对该同步状态进行原子操作实现对其值的修改。

状态信息通过 protected 类型的 getState，setState，compareAndSetState 进行操作

### 对资源的共享方式

**AQS 定义两种资源共享方式**

- Exclusive

  （独占）：只有一个线程能执行，如ReentrantLock又可分为公平锁和非公平锁：

  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的

- **Share**（共享）：多个线程可同时执行，如` CountDownLatch`、`Semaphore`、 `CyclicBarrier`、`ReadWriteLock` 我们都会在后面讲到。

### 模板方法模式

AQS底层用了模板方法模式

同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

1. 使用者继承 `AbstractQueuedSynchronizer` 并重写指定的方法。（这些重写方法很简单，无非是对于共享资源 state 的获取和释放）
2. 将 AQS 组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

**AQS 使用了模板方法模式，自定义同步器时需要重写下面几个 AQS 提供的模板方法：**

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

默认情况下，每个方法都抛出 `UnsupportedOperationException`。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS 类中的其他方法都是 final ，所以无法被其他类使用。

以 ReentrantLock 为例，state 初始化为 0，表示未锁定状态。A 线程 lock()时，会调用 tryAcquire()独占该锁并将 state+1。此后，其他线程再 tryAcquire()时就会失败，直到 A 线程 unlock()到 state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证 state 是能回到零态的。

再以 `CountDownLatch` 以例，任务分为 N 个子线程去执行，state 也初始化为 N（注意 N 要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后` countDown()` 一次，state 会 CAS(Compare and Swap)减 1。等到所有子线程都执行完后(即 state=0)，会 unpark()主调用线程，然后主调用线程就会从 `await()` 函数返回，继续后余动作。

### 组件总结

- **`Semaphore`(信号量)-允许多个线程同时访问：** `synchronized` 和 `ReentrantLock` 都是一次只允许一个线程访问某个资源，`Semaphore`(信号量)可以指定多个线程同时访问某个资源。
- **`CountDownLatch `（倒计时器）：** `CountDownLatch` 是一个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。
- **`CyclicBarrier`(循环栅栏)：** `CyclicBarrier` 和 `CountDownLatch` 非常类似，它也可以实现线程间的技术等待，但是它的功能比 `CountDownLatch` 更加复杂和强大。主要应用场景和 `CountDownLatch` 类似。`CyclicBarrier` 的字面意思是可循环使用（`Cyclic`）的屏障（`Barrier`）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。`CyclicBarrier` 默认的构造方法是 `CyclicBarrier(int parties)`，其参数表示屏障拦截的线程数量，每个线程调用 `await()` 方法告诉 `CyclicBarrier` 我已经到达了屏障，然后当前线程被阻塞。

### CountDownLatch使用场景

`CountDownLatch` 的作用就是 允许 count 个线程阻塞在一个地方，直至所有线程的任务都执行完毕。之前在项目中，有一个使用多线程读取多个文件处理的场景，我用到了 `CountDownLatch` 。具体场景是下面这样的：

我们要读取处理 6 个文件，这 6 个任务都是没有执行顺序依赖的任务，但是我们需要返回给用户的时候将这几个文件的处理的结果进行统计整理。

为此我们定义了一个线程池和 count 为 6 的`CountDownLatch`对象 。使用线程池处理读取任务，每一个线程处理完之后就将 count-1，调用`CountDownLatch`对象的 `await()`方法，直到所有文件读取完之后，才会接着执行后面的逻辑。

## 21. 死锁，如何避免

四个必要条件：

1. 互斥条件：一个资源只能被一个线程占用
2. 保持与等待条件：一个线程在等待获取另一个资源的时候，不会释放自己占有的资源
3. 不可剥夺条件：线程已经获得的资源在其未用完之前 不会被其他线程强行占有
4. 循环等待条件：若干线程之间形成一种头尾相接的循环等待资源

如何避免:

1. 破坏保持与等待条件：一次性申请所有的资源
2. 破坏不可剥夺条件：如果申请不到，那么就释放自己的资源
3. 破坏循环等待条件：按顺序申请资源，反序释放资源

## 22. java锁升级

无锁->偏向锁->轻量级锁->重量级锁

1. 检查当前的对象的对象头内是否是自己的线程ID，如果是，表示当前线程处于偏向锁状态
2. 如果不是，锁升级，用CAS来执行切换，新的线程根据现有的ThreadID，通知之前线程暂停，将Markword置为空
3. 两个线程把锁对象的hashcode复制到自己的用于存储锁的记录空间，开始CAS，把锁对象的markword内容修改为自己新建的记录空间的地址的方式竞争markword
4. 如果成功，则获得资源，失败则进入自旋
5. 自旋过程中 获得资源，整个状态依然是轻量级锁的状态
6. 若失败，进入重量级锁的状态，这个时候，自旋的进程阻塞，等待之前的线程执行完成唤醒自己

## 23. 线程池

### 为什么要使用线程池

1. 降低资源消耗，每次线程的创建和销毁都需要消耗资源

2. 加快响应速度，无需等待线程的建立

3. 提高线程的可管理性。线程是稀缺资源，无限制创建会消耗系统资源，还会降低系统稳定性

### 线程池的参数和执行流程

首先线程池有几个核心的参数概念：

1. 最大线程数maximumPoolSize
2. 核心线程数corePoolSize
3. 活跃时间keepAliveTime
4. 活跃时间的时间单位TimeUnit
5. 阻塞队列workQueue
6. 线程工厂threadFactory
7. 拒绝策略RejectedExecutionHandler

当提交一个新任务到线程池时，具体的执行流程如下：

1. 当我们提交任务，线程池会根据corePoolSize大小创建若干任务数量线程执行任务
2. 当任务的数量超过corePoolSize数量，后续的任务将会进入阻塞队列阻塞排队
3. 当阻塞队列也满了之后，将会新值线程数至maximumPoolSize，如果任务处理完成，这额外创建的线程在等待keepAliveTime后会被自动销毁
4. 如果达到maximumPoolSize，阻塞队列还是满的状态，那么将根据不同的拒绝策略对应处理

### 线程池的拒绝策略

AbortPolicy：直接丢弃任务，抛出异常

CallerRunsPolicy：调用execute函数的上层线程执行被拒绝的任务

DiscardOldesPolicy：丢弃最旧的

DiscardPolicy：丢弃 不抛异常

### 线程池的大小如何设置

**CPU 密集型**

- CPU 密集的意思是该任务需要大量的运算，而没有阻塞，CPU 一直全速运行。 
- CPU 密集型任务尽可能的少的线程数量，一般为 **CPU 核数 + 1 个**线程的线程池。 

**IO 密集型**

- 由于 IO 密集型任务线程并不是一直在执行任务，可以多分配一点线程数，如 **CPU \* 2** 
- 也可以使用公式：CPU 核数 / (1 - 阻塞系数)；其中阻塞系数在 0.8 ～ 0.9 之间。

## 24. Executors类可以创建哪些线程池

1. newSingleThread**Executor** 创建**一个单线程化**的线程池，只有一个线程的线程池，因此所有提交的任务是顺序执行，适用于一个一个任务执行的场景

2. newFixedThreadPool 拥有**固定线程数**的线程池，可控制线程最大并发数，超出的线程会在队列中等待。

3. newScheduledThreadPool 创建一个**可定期或者延时执行**任务的定长线程池，支持定时及周期性任务执行。 

4. newCachedThreadPool 线程池里有**很多线程需要同时执行**，老的可用线程将被新的任务触发重新执行，如果线程超过60秒内没执行，那么将被终止并从池中删除，适用执行很多短期异步的小程序或者负载较轻的服务

### 为什么不让使用Executors类的几种线程

FixedThreadPool 和 SingleThreadExecutor ： 允许请求的**队列⻓度**为 Integer.MAX_VALUE，可能堆积⼤量的请求，从⽽导致OOM(out fo memory)。 CachedThreadPool 和 ScheduledThreadPool ： 允许创建的**线程数量**为 Integer.MAX_VALUE，可能会创建⼤量线程，从⽽导致OOM。

## ReenTrantLock中condition有什么作用？condition的await和signal和Object的wait和notify有什么区别？

**Condition的强大之处在于，对于一个锁，我们可以为多个线程间建立不同的Condition**。

如果采用Object类中的wait(), notify(), notifyAll()实现的话，当写入数据之后需要唤醒读线程时，不可能通过notify()或notifyAll()明确的指定唤醒读线程，而只能通过notifyAll唤醒所有线程，但是notifyAll无法区分唤醒的线程是读线程，还是写线程。所以，通过Condition能够更加精细的控制多线程的休眠与唤醒。