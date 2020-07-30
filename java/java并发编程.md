[TOC]

# 概述
这章准备整理一下，java线程相关的东西





# 一、使用线程

java中使用线程的三种方式：

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



## Daemon

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺部分。

当所有非守护线程结束，程序终止，同时回杀死所有守护线程。



## sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException, 因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其他异常也同样需要再本地进行处理。

## yield()

对于静态方法 Thread.yield()的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其他线程来运行。



# 三、中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常，也会提前结束。



## InterruptedEception

通过调用一个线程的 interrupt()来中断线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出InterruptedException

























## wait(), notify(), notifyAll()

wait()，将一个想要获取某个对象锁的线程，放到等待池中等待。

notify(), 随机将等待池中的某个线程，唤醒，移动到锁池中，参与竞争锁

notifyAll()， 将等待池中的所有线程唤醒，移动到锁池中



**始终要在循环中调用wait方法；永远不要在循环之外调用它**。








## 线程的状态

## 创建线程的三种方式

## 中断

## 线程安全

## 锁

