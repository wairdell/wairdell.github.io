---
title: BlockingQueue
date: 2020-05-26 19:13:27
tags:
- 源码
- java
categories:
- 技术
---
### BlockingQueue
BlockingQueue 对应着中文名是"阻塞队列"，而什么是阻塞队列呢?当我们在生产者和消费者的使用场景中，如果消费者在队列没有数据的情况下会自定挂起，等到有数据了又会恢复运行;生产者在队列数据满的情况下会自定挂起，等到队列有位置时才把数据插进去。支持这两种操作的被称为阻塞队列。

### 常用方法
- boolean offer(E e)
  
    往队列中添加数据，不阻塞。如果队列没满，立即返回true； 如果队列满了，立即返回false

- void put(E e) throws InterruptedException;

    往队列中添加数据。如果队列满了则阻塞，直到队列中有空间可以添加数据

- boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;

    往队列中添加数据。如果队列满了则阻塞，直到队列中有空间可以添加数据或阻塞时间大于我们传入的时间

- E poll()

    从队列获取第一个数据(出队)，不阻塞，如果队列为空返回 `null`
- E take() throws InterruptedException;

    从队列获取第一个数据(出队)，如果队列为空则阻塞，直到可以获取到数据


- E poll(long timeout, TimeUnit unit) throws InterruptedException;

    从队列获取第一个数据(出队)，如果队列为空则阻塞，直到可以获取到数据或则阻塞时间大于我们传入的时间


### 常见 BlockingQueue

1. ArrayBlockingQueue
   
    一个由数组支持的有界阻塞队列。此队列按 FIFO（先进先出）原则对元素进行排序。队列的头部 是在队列中存在时间最长的元素。队列的尾部 是在队列中存在时间最短的元素。新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素。

    这是一个典型的“有界缓存区”，固定大小的数组在其中保持生产者插入的元素和使用者提取的元素。一旦创建了这样的缓存区，就不能再增加其容量。试图向已满队列中放入元素会导致操作受阻塞；试图从空队列中提取元素将导致类似阻塞。

    此类支持对等待的生产者线程和使用者线程进行排序的可选公平策略。默认情况下，不保证是这种排序。然而，通过将公平性 (fairness) 设置为 true 而构造的队列允许按照 FIFO 顺序访问线程。公平性通常会降低吞吐量，但也减少了可变性和避免了“不平衡性”。
2. LinkedBlockingQueue
   
    一个基于表链结构、范围任意的阻塞队列。此队列按 FIFO（先进先出）排序元素。队列的头部 是在队列中时间最长的元素。队列的尾部 是在队列中时间最短的元素。新元素插入到队列的尾部，并且队列获取操作会获得位于队列头部的元素。链接队列的吞吐量通常要高于基于数组的队列，但是在大多数并发应用程序中，其可预知的性能要低。

    可选的容量范围构造方法参数作为防止队列过度扩展的一种方法。如果未指定容量，则它等于 Integer.MAX_VALUE。除非插入节点会使队列超出容量，否则每次插入后会动态地创建链接节点。

3. PriorityBlockingQueue
   
    一个无界阻塞队列，它使用与类 PriorityQueue 相同的顺序规则，并且提供了阻塞获取操作。虽然此队列逻辑上是无界的，但是资源被耗尽时试图执行 add 操作也将失败（导致 OutOfMemoryError）。此类不允许使用 null 元素。依赖自然顺序的优先级队列也不允许插入不可比较的对象（这样做会导致抛出 ClassCastException）。

4. DelayQueue
   
    Delayed 元素的一个无界阻塞队列，只有在延迟期满时才能从中提取元素。该队列的头部 是延迟期满后保存时间最长的 Delayed 元素。如果延迟都还没有期满，则队列没有头部，并且 poll 将返回 null。当一个元素的 getDelay(TimeUnit.NANOSECONDS) 方法返回一个小于等于 0 的值时，将发生到期。即使无法使用 take 或 poll 移除未到期的元素，也不会将这些元素作为正常元素对待。例如，size 方法同时返回到期和未到期元素的计数。此队列不允许使用 null 元素。

5. SynchronousQueue

    一种阻塞队列，其中每个插入操作必须等待另一个线程的对应移除操作 ，反之亦然。同步队列没有任何内部容量，甚至连一个队列的容量都没有。不能在同步队列上进行 peek，因为仅在试图要移除元素时，该元素才存在；除非另一个线程试图移除某个元素，否则也不能（使用任何方法）插入元素；也不能迭代队列，因为其中没有元素可用于迭代。队列的头 是尝试添加到队列中的首个已排队插入线程的元素；如果没有这样的已排队线程，则没有可用于移除的元素并且 poll() 将会返回 null。对于其他 Collection 方法（例如 contains），SynchronousQueue 作为一个空 collection。此队列不允许 null 元素。

6. LinkedBlockingDeque
   
    一个基于链表结构、任选范围的阻塞双端队列。

    可选的容量范围构造方法参数是一种防止过度膨胀的方式。如果未指定容量，那么容量将等于 Integer.MAX_VALUE。只要插入元素不会使双端队列超出容量，每次插入后都将动态地创建链接节点。

    大多数操作都以固定时间运行（不计阻塞消耗的时间）。异常包括 remove、removeFirstOccurrence、removeLastOccurrence、contains、iterator.remove() 以及批量操作，它们均以线性时间运行。
   

### 实现原理

通过比较简单的 ArrayBlockingQueue，来讲讲实现原理。
- 用 `items` 对象数组来保存元素。在所有添加或删除的操作中用 `lock` 重入锁对象保证线程安全。用 notEmpty 和 notFull 来完成入队和出队时的阻塞操作。
- 在入队的时候判断当前的元素是不是大于最大的。如果最大对于 `boolean offer(E e)` 方法直接返回 false。而对于 `void put(E e)` 方法则使用条件对象 notFull 的 await 方法等待(阻塞线程)。当有元素出队时则用 条件对象 notFull 的 signal 方法激活等待的线程。
- 在出队的时候判断当前的元素是否为空。为空时对于`E poll()` 方法直接返回 null。而对于 `E take() throws InterruptedException;` 方法则使用条件对象 notEmpty 的 await 方法等待(阻塞线程)。当有元素入队时则用 条件对象 notEmpty 的 signal 方法激活等待的线程。
```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {

    /** 重入锁，入队和出队用的同一个 */
    final ReentrantLock lock;

    /** 条件对象,出队时队列为空时用这个进行阻塞操作 */
    private final Condition notEmpty;

    /** 条件对象,入队时队列满时用这个进行阻塞操作 */
    private final Condition notFull;

    /** 用对象数字保存队列的元素 */
    final Object[] items;

    /** 队列中元素的数量 */
    int count;

    /** 队首元素下标，获取数据的位置 */
    int takeIndex;

    /** 对尾元素下标，添加数据的位置 */
    int putIndex;

    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }


    public void put(E e) throws InterruptedException {
        Objects.requireNonNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            //队列满时，用 notFull.await() 等待(阻塞线程)
            while (count == items.length)
                notFull.await();
            //入队
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }

    public boolean offer(E e) {
        Objects.requireNonNull(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            //队列满时，直接返回false
            if (count == items.length)
                return false;
            else {
                //入队
                enqueue(e);
                return true;
            }
        } finally {
            lock.unlock();
        }
    }

    public boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException {

        Objects.requireNonNull(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length) {
                if (nanos <= 0L)
                    return false;
                //队列满时，用 notFull.awaitNanos() 等待(阻塞线程) ，如果超过，则退出等待
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }
    }

    /** 入队 */
    private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length) putIndex = 0;
        count++;
        //激活正在等待取元素的线程(等待的可能有多个，这里是随机某一个)
        notEmpty.signal();
    }

     
     public E poll() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            //如果队列为空，直接返回null
            return (count == 0) ? null : dequeue();
        } finally {
            lock.unlock();
        }
    }

    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            //如果队列为空，用 notEmpty.await() 等待(阻塞线程)
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

    public E poll(long timeout, TimeUnit unit) throws InterruptedException {
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0) {
                if (nanos <= 0L)
                    return null;
                //如果队列为空，用 notEmpty.awaitNanos() 等待(阻塞线程)，如果超过，则退出等待
                nanos = notEmpty.awaitNanos(nanos);
            }
            return dequeue();
        } finally {
            lock.unlock();
        }
    }


    /** 出队 */
    private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length) takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        //激活正在等待添加元素的线程(等待的可能有多个，这里是随机某一个)
        notFull.signal();
        return x;
    }    

}
```