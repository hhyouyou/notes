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

![Executor类继承关系图](http://image-djx.test.upcdn.net/md/notes/Executor%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB%E5%9B%BE.png)













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

