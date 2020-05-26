---
title: ThreadPoolExecutor
date: 2020-05-26 23:02:05
tags:
---
### ThreadPoolExecutor
```
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
}
```
1. corePoolSize
   
    核心线程数，当有新任务提交的时候，会检测当前线程池内的线程数是否小于 corePoolSize。小于这新建一个线程将任务交给新线程处理；否则就将任务添加到 workQueue 中
2. maximumPoolSize
   
   线程池内最大的线程数。
3. keepAliveTime
   
   空闲线程存活的时间。核心线程无效。
4. workQueue
   
   当运行的线程数大于等于 corePoolSize 时，会将任务先存到 workQueue
5. threadFactory
   
   线程工厂，ThreadPoolExecutor 每次要新建一个新的线程会用这个对象新建。
6. handler
   
   拒绝策略。当任务的数量超过最大线程数(maximumPoolSize) + 任务队列(workQueue)最大的容量时会调用这个对象处理这种情况。

### 常用实现

Executors 为我们提供一些比较常用的线程池。大部分的使用场景我们用下面这些就足够了。

- FixedThreadPool
    ```
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
    }
    ```
    固定大小的线程池,核心线程和最大线程数一样，只有核心线程，没有多余的线程所以 keepAliveTime 为0

- CachedThreadPool
    ```
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
    }
    ```
    缓存线程池，没有核心线程，所有的线程自动回收，这里用的阻塞队列是不会存储元素的，每一次添加任务都会立马执行。

- SingleThreadExecutor
    ```
    public static ExecutorService newSingleThreadExecutor() {
        return new Executors.FinalizableDelegatedExecutorService(new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue()));
    }
    ```
    单一线程池。核心线程和最大线程都为1。这里 FinalizableDelegatedExecutorService 包了一次是为了防止相关配置被更改。相关的方法还是通过委托模式调用传进去的 ThreadExecutor 。

- ScheduledThreadPool 
  ```
    public static ScheduledExecutorService newScheduledThreadPool() {
        return new ScheduledThreadPoolExecutor(1);
    }
  ```
  一个能定时和周期性任务的线程池