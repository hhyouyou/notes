[TOC]

# 概述
这章准备整理一下，java线程相关的东西





# 一、使用线程

Java中使用线程的三种方式：

* 继承 Thread 类
* 实现 Runnable 接口
* 实现 Callable 接口

实现Runnable和Callable 接口的类只能当作一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以理解为任务是通过线程驱动从而执行的。

## 继承 Thread 类

同样也需要实现 run() 方法， 因为 Thread 类也实现了 Runnable 接口。

当我们调用start() 方法启动一个线程时， 虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。 

```java
 public static class Thread1 extends Thread {
     @Override
     public void run() {
         System.out.println(this.getClass().getName());
         super.run();
     }
 }
```

## 实现Runnable接口

需要实现 Runnable 接口中的 run() 方法。

```java
 public static class Thread2 implements Runnable {
     @Override
     public void run() {
         System.out.println(this.getClass().getName());
     }
 }
```

## 实现Callable接口

和Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

```java
public static class Thread3 implements Callable<String> {
    @Override
    public String call() throws Exception {
        return this.getClass().getName();
    }
}
```



运行结果

```java
  public static void main(String[] args) {
      // 1.继承 Thread
      Thread thread1 = new Thread(new Thread1(), "thread1");
      thread1.start();
      // 2.实现 Runnable
      Thread thread2 = new Thread(new Thread2(), "thread2");
      thread2.start();
      // 3.实现 Callable
      FutureTask<String> stringFutureTask = new FutureTask<>(new Thread3());
      Thread thread3 = new Thread(stringFutureTask, "thread3");
      thread3.start();
      try {
          String s = stringFutureTask.get();
          System.out.println(s);
      } catch (InterruptedException | ExecutionException e) {
          e.printStackTrace();
      }
  }
```

## 实现接口  和  继承 Thread 类的对比

实现接口相对更好：

* Java 支持单继承，多实现。实现接口更灵活
* 继承整个 Thread类开销更大



# 二、基础线程机制

## Executor

Executor 管理多个异步任务的执行，而无需我们显式的管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor :

* CachedThreadPool: 一个任务创建一个线程
* FixedThreadPool: 所有任务只能使用固定大小的线程池
* SingleThreadExecutor: 只有一个线程的线程池

### Executor 接口 及其 继承关系

![Executor类继承关系图](http://image-djx.test.upcdn.net/md/notes/Executor%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB%E5%9B%BE.png)

* `Executor`：是一个接口，定义了一个接受Runnable对象的方法executor

  ```java
  public interface Executor { void execute(Runnable command);}
  ```

* `ExecutorService`: 是一个比`Executor` 使用更加广泛的子类接口，它提供了生命周期管理的方法，以及可以跟踪一个或多个异步任务执行状况返回`Future`的方法。shutdown()、submit()、invokeAll()等

* `AbstractExecutorService`: `ExecutorService`执行方法的默认实现`

* `ScheduledExecutorService`: 一个可定时调度任务的接口

* `ScheduledThreadPoolExecutor`: `ScheduledExecutorService`的实现，一个可定时调度任务的线程池。(这里还用了装饰者模式，装饰定时任务？没看懂)

* `ThreadPoolExecutor` : 表示一个线程池，可以通过`Executors`的静态工厂方法来创建一个拥有特定功能的线程池，并返回一个 `ExcutorService`对象。

以上类、接口都在 `java.util.concurrent`包中，是JDK并发包的核心类。 其中 `ThreadPoolExecutor` 表示一个线程池。 `Executors` 类则扮演着线程池工厂的角色，通过`Executors`可以取得一个拥有特定功能的线程池。从UML图中也可以看到，`ThreadPoolExecutor` 类实现了` Executor` 接口， 因此，任何Runnable的对象都可以被`ThreadPoolExecutor `线程池调度。

## new ThreadPoolExecutor(...)

> 线程资源必须通过线程池创建，不允许在应用内自行显示创建。——《阿里巴巴Java手册》

### why

1. 降低开销。创建和销毁线程会产生很大的系统开销。
2. 易复用和管理。将线程都放在一个公用的池子中，便于统一管理，也便于复用。
3. 解耦。将线程的创建和销毁从业务代码中分离出来。便于维护。



### 构造方法

```java
public ThreadPoolExecutor(int corePoolSize,// 核心线程数
                          int maximumPoolSize,// 线程池最大线程数
                          long keepAliveTime,// 空闲线程存活时间
                          TimeUnit unit,// 时间单位
                          BlockingQueue<Runnable> workQueue,// 阻塞对列
                          ThreadFactory threadFactory,// 线程工厂
                          RejectedExecutionHandler handler) {// 拒绝策略
    if (corePoolSize < 0 ||maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize || keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?null : AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

#### corePoolSize

线程池中的核心线程数。提交一个任务到线程池时，会创建一个线程来执行任务，直到线程数等于corePoolSize就不再创建，此时继续提交的任务则会保存到阻塞对列中。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有线程。

#### maximumPoolSize

线程池最大线程数，如果当前阻塞对列满了，继续提交任务，若当前线程数小于maximumPoolSize则创建新的线程执行任务。如果使用了无界的阻塞对列这个参数就没用了。

#### KeepAliveTime

非核心线程空闲时保持存活时间，即当线程没有任务执行时间时，继续存活的时间。

#### unit

线程保持活跃时间单位

#### BlockingQueue workQueue

用于保存等待执行任务的阻塞队列。[BlockingQueue](#常用的阻塞队列)

#### ThreadFactory

创建线程工厂

#### RejectedExecutionHandler handler：饱和策略

当队列和最大线程池都满了之后的饱和策略

- CallerRunsPolicy : 调用线程处理任务
- AbortPolicy : 抛出异常
- DiscardPolicy : 直接丢弃
- DiscardOldestPolicy : 丢弃队列中最老的任务，执行新任务


## Daemon

守护线程是程序运行时在 提供服务的线程，不属于程序中不可或缺部分。

当所有非守护线程结束，程序终止，同时回杀死所有守护线程。



## sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException, 因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其他异常也同样需要再本地进行处理。

## yield()

对于静态方法 Thread.yield()的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其他线程来运行。



# 三、中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常，也会提前结束。

## InterruptedEception

通过调用一个线程的 interrupt()来中断线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出InterruptedException，从而提前结束该线程。但是不能中断I/O 阻塞和 synchronized 锁阻塞。

下面先启动一个线程，然后在sleep的时候，中断它，就会抛出一个 InterruptedException，从而结束线程，不执行后面的语句。

```java
public static void main(String[] args) {
    Thread thread = new Thread(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });

    thread.start();
    thread.interrupt();
    System.out.println("main run");
}
```

```java
main run
java.lang.InterruptedException: sleep interrupted
at java.lang.Thread.sleep(Native Method)
at TestInterruptedException.lambda$main$0(TestInterruptedException.java:17)
at java.lang.Thread.run(Thread.java:748)
```



## interrupted()

如果一个线程的 run() 方法执行一个无循环，并且没有执行sleep()等，会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使该线程提前结束。

但是调用 interrupt()方法会设置线程的中断标记，此时调用 inerrupted() 方法会返回 true。 因此可以在循环体重使用interrupted()方法来判断线程是否处于中断状态的，从而提前结束线程。

也就是说，这个方法在线程正常运行时，无法停止运行，只能给这个线程打个标记，申请中断了。

```java
 public static void main(String[] args) {
     Thread thread = new Thread(() -> {
         while (!Thread.interrupted()){
             System.out.println("123");
         }
         System.out.println("结束");
     });

     thread.start();
     thread.interrupt();
 }
```



## Executor 的中断操作

调用 Executor 的  shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是shutdownNow()方法，则相当于调用每个线程的interrupt()方法。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    executorService.shutdownNow();
    System.out.println("main run");
}
```

```java
main run
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at ShutdownExecutor.lambda$main$0(ShutdownExecutor.java:16)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```

如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?>对象，通过调用该对象的cancel(true)方法就可以中断该线程。

```java
// 中断单个线程
Future<?> submit = executorService.submit(() -> {});
submit.cancel(true);
```



# 四、互斥同步

Java提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的synchronized， 而另一个是 JDK 实现的 ReentrantLock 。

## synchronized

1. 同步一个代码块

   ```java
   public void f(){
       synchronized(this){
           // ...
       }
   }
   ```

   它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会同步。

   对于以下代码块，使用ExecutorService执行了两个线程。由于test1()调用的是同一个对象的同步代码块，所以是同步执行的，当一个线程进入时，另一个线程就在等待。而test2()调用的是不同对象的同步代码块，所以两个线程就不同步了。

   ```java
   public static void main(String[] args) {
       test1(); // 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 
       test2(); // 0 1 2 3 4 5 6 7 8 9 10 0 1 2 3 4 11 5 6 7 8 9 12 13 14 10 11 12 13 14 
   }
   
   private static void test2() {
       SynchronizedDemo1 demo1 = new SynchronizedDemo1();
       SynchronizedDemo1 demo2 = new SynchronizedDemo1();
       ExecutorService executorService = Executors.newCachedThreadPool();
       executorService.execute(() -> demo1.f1());
       executorService.execute(() -> demo2.f1());
       executorService.shutdown();
   }
   
   private static void test1() {
       SynchronizedDemo1 synchronizedDemo1 = new SynchronizedDemo1();
       ExecutorService executorService = Executors.newCachedThreadPool();
       executorService.execute(() -> synchronizedDemo1.f1());
       executorService.execute(() -> synchronizedDemo1.f1());
       executorService.shutdown();
   }
   
   static class SynchronizedDemo1 {
       public void f1() {
           synchronized (this) {
               for (int i = 0; i < 20; i++) {
                   System.out.print(i + " ");
               }
           }
       }
   }
   ```

2. 同步一个方法

   和同步代码块一样，作用于一个方法

   ```java
   public synchronized void func () {
       // ...
   }
   ```

3. 同步一个类

   ```java
   public void func() {
       synchronized (SynchronizedDemo.class) {
           // ...
       }
   }
   ```

   作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会镜进行同步。
   
   ```java
   public static void main(String[] args) {
       test3(); // 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
   }
   private static void test3() {
       SynchronizedDemo2 demo1 = new SynchronizedDemo2();
       SynchronizedDemo2 demo2 = new SynchronizedDemo2();
       ExecutorService executorService = Executors.newCachedThreadPool();
       executorService.execute(() -> demo1.f1());
       executorService.execute(() -> demo2.f1());
       executorService.shutdown();
   }
   static class SynchronizedDemo2 {
       public void f1() {
           synchronized (SynchronizedDemo2.class) {
               for (int i = 0; i < 100; i++) {
                   System.out.print(i + " ");
               }
           }
       }
   }
   ```
   
4. 同步一个静态方法

   静态方法也是作用于整个类的。

   ```java
   public synchronized static void fun() {
       // ...
   }
   ```

## ReentrantLock

ReentrantLock是 java.util.concurrent (J.U.C) 包中的锁。

```java
public static class ReentrantLockDemo {
    private Lock lock = new ReentrantLock();
    public void test() {
        lock.lock();
        try {
            for (int i = 0; i < 100; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock();
        }
    }
}
```

```java

 public static void main(String[] args) {
     ExecutorService executorService = Executors.newCachedThreadPool();
     ReentrantLockDemo reentrantLockDemo = new ReentrantLockDemo();
     executorService.execute(() -> reentrantLockDemo.test());
     executorService.execute(() -> reentrantLockDemo.test());
 }
```

```java
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```



## 比较

1. 锁的实现

   synchronized是JVM实现的， 而ReentrantLock 是JDK 实现的。

2. 性能

   差不多。

3. 等待可中断

   当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

   ReentrantLock可以中断，而synchronized不行。

4. 公平锁

   公平锁，是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

   synchronized中的锁是非公平的， ReentrantLock默认情况下是非公平的，也可以是公平的。

5. 锁绑定多个条件

   一个ReentrantLock 可以同时绑定多个Condition 对象

## 使用选择

除非需要使用ReentrantLock的高级功能，否则优先使用synchronized。这是因为，synchronized 是 JVM实现的一种锁机制， JVM 原生的支持它，而ReentrantLock不是所有的JDK版本都支持。并且使用 synchronized 不用担心没有实放锁而导致死锁的问题，因为JVM会确保锁的释放。



# 五、线程之间的协作

当多个线程可以一起工作去解决某个问题的时候，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

## join()

在线程中调用另外一个线程的join()方法， 会将当前线程挂起，而不是忙等待，直到目标线程结束。

所以下面这个demo , 虽然b先启动，但是因为在 b线程中调用了a线程的join()方法，所以b线程会等a线程结束后才继续执行。

```java
public class JoinDemo {

    public static void main(String[] args) {
        A a = new A();
        B b = new B(a);

        b.start();
        a.start();
    }

    private static class A extends Thread {
        @Override
        public void run() {
            System.out.println("a");
        }
    }

    private static class B extends Thread {
        private A a;

        public B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("b");
        }
    }
}
```

```java
a
b
```

## wait(), notify(), notifyAll()

这几个方法都属于 Object ，而不是 Thread。

在使用wait() 挂起时，线程会释放锁。这是因为，如果没有释放锁，那么其他线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

* wait()，将一个想要获取某个对象锁的线程，放到等待池中等待。
* notify(), 随机将等待池中的某个线程，唤醒，移动到锁池中，参与竞争锁
* notifyAll()， 将等待池中的所有线程唤醒，移动到锁池中

只有在同步方法或者是同步代码块中使用，否则会在运行时抛出 IllegalMonitorStateException

**始终要在循环中调用wait方法；永远不要在循环之外调用它**。

## awiat() 、signal() 、signalAll()

java.util.concurrent 类库中提供了Condition 类来实现线程之间的协调，可以再Condition 上调用 await()方法使线程等待，其他线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait()这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用Lock来获取一个Condition对象。

然后调Condition对象的 await()和signalAll()方法

```java
public class AwaitSignalDemo {

    private Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    private void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
        System.out.println();
    }

    private void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        AwaitSignalDemo awaitSignalDemo = new AwaitSignalDemo();
        executorService.execute(awaitSignalDemo::after);
        executorService.execute(awaitSignalDemo::before);
    }
}
```

```java
before
after
```



# 六、线程的状态

线程状态，一个线程只能处于一种状态，并且这里的线程状态特指**Java虚拟机的线程状态**，不能反映线程在特定操作系统下的状态。

## 新建(NEW)

创建后尚未启动

## 可运行（RUNABLE）

正在Java虚拟机中运行。但是在操作系统层面，他可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就 进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。

## 阻塞（BLOCKED）

请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其他线程已经占用了该 monitor lock，所以处于阻塞状态。要结束该状态，从而进入 RUNABLE 需要其他线程释放 monitor lock。

## 无限期等待（WAITING）

等待其他线程显式的唤醒。

阻塞和等待的区别在于，阻塞是被动的，被阻塞的线程是在等待获取 monitor lock。而等待是主动的，通过 Object.wait() 等方法进入。

| 进入方法                                   | 退出方法                              |
| ------------------------------------------ | ------------------------------------- |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify()  、Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 等待被调用的线程执行完毕              |
| LockSupport.park() 方法                    | LockSupport.unpark(Thread)            |

## 限期等待（TIMED_WAITING）

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒

| **进入方法**                         | 退出方法                                      |
| ------------------------------------ | --------------------------------------------- |
| Thread.sleep() 方法                  | 时间结束                                      |
| 设置Timeout参数的Object.wait()方法   | 时间结束、Object.notify()、Object.notifyAll() |
| 设置了Timeout参数的Thread.join()方法 | 时间结束、被调用的线程执行完毕                |
| LockSupport.parkNanos()方法          | LockSupport.unpark(Thread)                    |
| LockSupport.parkUtil() 方法          | LockSupport.unpark(Thread)                    |

调用 Thread.sleep()方法使线程进入限期等待状态时，常用”使一个线程随眠“进行描述。调用Object.wait()方法使线程进入限期等待或者无限期等待时，常用”挂起一个线程“进行描述。睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

## 死亡（TERMINATED）

可以是线程结束任务之后自己结束，或者产生异常而结束



# 七、 J.U.C -AQS

java.util.concurrent (J.U.C) 大大提高了并发性能，AQS被认为是 J.U.C 的核心。

## CountDownLatch

倒数计时，用来控制一个或者多个线程等待多个线程。

维护了一个计数器cnt，每次调用 countDown() 方法会人计数器的值减1，减到0 的时候，那些因为调用await() 方法而在等待的线程就会被唤醒。

```java
public class CountDownLatchDemo {

    public static void main(String[] args) throws InterruptedException {

        ExecutorService executorService = Executors.newCachedThreadPool();
        CountDownLatch countDownLatch = new CountDownLatch(5);

        executorService.execute(() -> {
            try {
                System.out.println("ready");
                countDownLatch.await();
                System.out.println("end");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        for (int i = 0; i < 5; i++) {
            executorService.execute(() -> {
                System.out.print("one thread count ..");
                countDownLatch.countDown();
            });
        }

        executorService.shutdown();
    }
    
}
```

当执行countDownLatch.countDown() = 设定次数时，调用 countDownLatch.await()的线程就会被唤醒

```java
ready
one thread count ..
one thread count ..
one thread count ..
one thread count ..
one thread count ..
end
```

## CyclicBarrier

循环屏障，用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法后，计数器会减1，并进行等待，直到计数器为0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和CountDownLatch 的一个区别是，CyclicBarrier 的计数器是通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier 有两个构造 函数， 其中parties 指示计数器的初始值，barrierAction在所有线程都到达屏障的时候会执行一次。

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}
```

```JAVA
public CyclicBarrier(int parties) {
    this(parties, null);
}
```

demo 如下

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            executorService.execute(() -> {
                try {
                    System.out.println("ready...");
                    cyclicBarrier.await();
                    System.out.println("go!");
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
        }
        executorService.shutdown();
    }
}
```

```java
ready...
ready...
ready...
ready...
ready...
go!
go!
go!
go!
go!
```



## Semaphore

semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

以下demo模拟来了对某个服务的并发请求，每次只能有三个客户端同时访问，请求总和为10。

```java
public class SemaphoreDemo {

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < 10; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    System.out.println("剩余数量" + semaphore.availablePermits());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }
        executorService.shutdown();
    }
}
```

```java
剩余数量2
剩余数量0
剩余数量1
剩余数量0
剩余数量2
剩余数量2
剩余数量1
剩余数量2
剩余数量0
剩余数量1
```



# 八、J.U.C - 其他组件

## FutureTask

在介绍Callable时 我们知道它可以有返回值，返回值通过 Future 进行封装。FutureTask 实现了RunnableFuture 接口，该接口继承自 Runnable和 Future 接口，这使得 FutureTask 既可以当做一个任务执行，也可以有返回值。

```java
public class FutureTask<V> implements RunnableFuture<V>
```

```java
public interface RunnableFuture<V> extends Runnable, Future<V>	
```

FutureTask可用于异步获取执行结果，或者是取消执行任务的场景。当一个计算任务需要执行很长时间时，那么就可以用FutureTask 来封装这个任务，主线程在完成自己的任务之后再去获取结果。

```java
public class FutureTaskDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> vFutureTask = new FutureTask<>(() -> {
            int result = 0;
            for (int i = 0; i < 100; i++) {
                Thread.sleep(10);
                result = result + i;
            }
            return result;
        });

        Thread thread = new Thread(vFutureTask);
        thread.start();

        Thread otherThread = new Thread(() -> {
            System.out.println("other task is running...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        otherThread.start();
        System.out.println(vFutureTask.get());
    }
}
```

```java
other task is running...
4950
```



## BlockingQueue

java.util.concurrent.BlockingQueue 接口有以下阻塞队列实现

* FIFO 队列 ：LinkedBlockingQueue、ArrayBlockingQueue(固定长度)
* 优先级队列：PriorityBlockingQueue

提供了阻塞的take()和 put() 方法： 

如果队列为空 take()将阻塞，直到队列中有内容。

如果队列为满put()将阻塞，直到队列中有空闲位置。

```java
public class BlockingQueueDemo {

    public static BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(5);

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        Thread producter = new Thread(() -> {
            String code = RandomStringUtils.randomNumeric(6);
            try {
                blockingQueue.put(code);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("producer(" + code + ")..");
        });

        Thread consumer = new Thread(() -> {
            String take = null;
            try {
                take = blockingQueue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consumer(" + take + ")...");
        });

        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(producter);
        executorService.execute(producter);
        executorService.execute(consumer);
        executorService.execute(consumer);
        executorService.execute(consumer);

        executorService.shutdown();
    }
}
```

队列中没有元素时，消费者会等待生产者生产，阻塞在这里。

```java
Connected to the target VM, address: '127.0.0.1:0', transport: 'socket'
consumer(569389)...producer(170545)..consumer(170545)...producer(569389)..
```

没有消费完，主线程还是正常结束

```java
Connected to the target VM, address: '127.0.0.1:0', transport: 'socket'
consumer(692109)...consumer(884841)...producer(124762)..producer(884841)..producer(692109)..
Disconnected from the target VM, address: '127.0.0.1:0', transport: 'socket'
```

### 常用的阻塞队列：

1. **SynchronousQueue **:  同步移交队列，适用于非常大的或者无界的线程池，可以避免任务排队。SynchronousQueue 队列接收到任务后，会直接将任务从生产者移交给工作线程，这种移交机制高校。只有当线程池是无界的或者可以拒绝任务时，使用SynchronousQueue队列才有意义。要将一个元素放入SynchronousQueue，就需要有另一个线程才等待接收者这个元素，如果没有线程在等待，要么新建一个线程，要么拒绝掉。`newCachedThreadPool`默认使用的就是这种同步移交队列。吞吐量高约 LinkedBlockingQueue。
2. **LinkedBlockingQueue**: 基于链表结构的阻塞队列，FIFO 原则排序。当任务提交过来时，如果线程数小于核心线程数则等待，大于的时候就进入工作队列进行等待。LinkedBlockingQueue队列没有最大值限制，只要任务数超过核心线程，都会被添加到队列中，这就会导致，总线程数永远不会超过核心线程数，所以这个时候 maximumPoolSize就没意义了。 `newFixedThreadPool`和`newSingleThreadPool`默认使用的是 无界的 LinkedBlockingQueue队列。吞吐量高于 ArrayBlockingQueue
3. **ArrayBlockingQueue** : 基于数组结构的 有界阻塞队列，可以设置队列上限值，FIFO原则排序。当任务提交过来时，如果线程数小于核心线程数则等待；大于的时候就进入工作队列进行等待；如果队列也满了，则创建非核心线程执行任务；如果总线程达到了 maximumPoolSize，则根据饱和策略去拒绝。
4. **DelayQueue** :  延迟队列，队列中的元素必须实现 Delayed 接口。当任务体骄傲时，入队列只有达到指定的延时时间，才会执行任务。
5. **PriorityBlockingQueue**：优先级阻塞队列，根据优先级执行任务，优先级是通过自然排序或者是Comparator定义实现。





## ForkJoin

主要用于并行计算中，和MapReduce 原理类似，都是把大的计算任务拆分成多个小任务并行计算。

```java
public class ForkJoinDemo extends RecursiveTask<Integer> {

    private final int threshold = 5;
    private int first;
    private int last;

    public ForkJoinDemo(int first, int last) {
        this.first = first;
        this.last = last;
    }

    @Override
    protected Integer compute() {
        int result = 0;
        if (last - first <= threshold) {
            // 直接计算
            for (int i = first; i <= last; i++) {
                result += i;
            }
        } else {
            // 拆分任务
            int middle = first + (last - first) / 2;
            ForkJoinDemo leftTask = new ForkJoinDemo(first, middle);
            ForkJoinDemo rightTask = new ForkJoinDemo(middle + 1, last);
            leftTask.fork();
            rightTask.fork();
            result = leftTask.join() + rightTask.join();
        }
        return result;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinDemo forkJoinDemo = new ForkJoinDemo(1, 10000);
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> submit = forkJoinPool.submit(forkJoinDemo);
        System.out.println(submit.get());
    }
}
```

ForkJoin 使用ForkJoinPool 来启动， 是一个特殊的线程池，线程数量取决于CPU核数

```
public class ForkJoinPool extends AbstractExecutorService
```

ForkJoinPool 实现了工作窃取算法（workSteal）来提高CPU 的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。工作窃取算法，允许空闲的线程从其他线程的双端队列中窃取一个任务来执行。窃取的任务必须是最晚的任务，避免和队列所属线程发生竞争。



# 九、线程不安全示例

多个线程对同一个共享数据进行访问，而不采取同步操作的话，那么结果很容易出问题。

```java
public class ThreadUnsafeDemo {

    public static class UnsafeIncrementDemo {
        private int cnt = 0;

        public void add() {
            cnt++;
        }
        public int get() {
            return cnt;
        }
    }

    public static class SafeIncrementDemo {
        private AtomicInteger cnt = new AtomicInteger(0);

        public void add() {
            cnt.incrementAndGet();
        }
        public int get() {
            return cnt.get();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        UnsafeIncrementDemo unsafeIncrementDemo = new UnsafeIncrementDemo();
        SafeIncrementDemo safeIncrementDemo = new SafeIncrementDemo();
        CountDownLatch countDownLatch = new CountDownLatch(1000);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < 1000; i++) {
            executorService.execute(() -> {
                unsafeIncrementDemo.add();
                safeIncrementDemo.add();
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println("unsafeIncrementDemo:" + unsafeIncrementDemo.get());
        System.out.println("safeIncrementDemo:" + safeIncrementDemo.get());
        executorService.shutdown();
    }
}
```

```java
unsafeIncrementDemo:986
safeIncrementDemo:1000
```



# 十、Java 内存模型

Java 内存模型，试图屏蔽各种硬件和操作系统的内存访问差异，以实现让Java程序在各个平台下都能达到一致的内存访问效果。

## 主内存 和 工作内存

处理上的寄存器比内存快了多个数量级，所以引入了高速缓存。由此引发了新的问题，缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存数据可能会不一致，需要一些协议来解决问题。 

开发中，所有的变量都存在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能通过操作工作内存中的变量，不同线程见的变量值传递需要通过主内存。

## 内存间的交互操作

Java的内存中定义了8个操作，来完成工作内存和主内存之间的交互操作:

* read : 把一个变量从主内存传输到工作内存中
* load : 在read之后执行，把read得到的值放入工作内存的变量副本中。
* use : 把工作内存中的一个变量的值传递给执行引擎
* assign : 把一个从执行引擎接收到的值赋给工作内存的变量
* store：把工作内存的一个变量的值传送到主内存中
* write：在store之后执行，把store得到的值写放入主内存的变量中。
* lock：作用于主内存的变量
* unlock：同上

## 内存模型三大特性

### 1.原子性

Java 内存模型保证了read、load、use、assign、store、write、lock和unlock操作具有原子性，例如对一个int变量执行assign赋值操作，这个操作就是原子性的。但是Java内存模型允许虚拟机将没有被volatile修饰的64位数据（long,double）的读写操作划分为两次32位的操作来进行，既load、store、read和write操作可以不具备原子性。

### 2.可见性

### 3.有序性

## 先行先发原则







































































