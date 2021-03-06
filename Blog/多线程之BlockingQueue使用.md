# BlockingQueue


简介：BlockingQueue是一个先进先出的阻塞队列。常用于生产者消费者的场景下。

BlockingQueue继承了Queue接口，所以在了解BlockingQueue接口之前，先了解一下Queue接口有哪些抽象方法。

```java
public interface Queue<E> extends Collection<E> {

    boolean add(E e);////将一个非空非null元素插入到该队列，如果插入成功返回true,不成功抛出异常

    boolean offer(E e);////将一个非空非null元素插入到该队列，如果插入成功返回true,不成功返回false

    E remove();//删除当前队列的头部元素，并返回头部元素,如果为空，抛出异常

    E poll();//删除当前队列的头部元素，并返回头部元素,如果为空，返回null

    E element();//获取当前队列的头部元素，如果为空，抛出异常

    E peek();//获取当前队列的头部元素，如果为空，返回null
}
```


BlockingQueue接口继承Queue接口方法基础上，还额外添加了如下几个方法：
```java
public interface BlockingQueue<E> extends Queue<E> {

    //将给定元素设置到队列中，如果设置成功返回true, 否则返回false。
    boolean add(E e);

    //将给定的元素设置到队列中，如果设置成功返回true, 否则返回false. 如果e值为空则抛出空指针异常。
    boolean offer(E e);

    //将元素设置到队列中，如果队列中没有多余的空间，该方法会一直阻塞，直到队列中有多余的空间。
    void put(E e) throws InterruptedException;

    //将元素设置到队列中，如果队列中没有多余的空间，该方法会一直阻塞，直到队列中有多余的空间。
    boolean offer(E e, long timeout, TimeUnit unit)
            throws InterruptedException;

    //从队列中获取值。如果队列中没有值，线程会一直阻塞，直到队列中有值，并且该方法取得了该值。
    E take() throws InterruptedException;

    //在给定的时间里，从队列中获取值，如果没有取到会抛出异常。
    E poll(long timeout, TimeUnit unit)
            throws InterruptedException;

    //获取队列中剩余的空间
    int remainingCapacity();

    //从队列中移除指定的值。
    boolean remove(Object o);

    //判断队列中是否拥有该值。
    public boolean contains(Object o);

    //将队列中值，全部移除，并发设置到给定的集合中。
    int drainTo(Collection<? super E> c);

    //指定最多数量限制将队列中值，全部移除，并发设置到给定的集合中。
    int drainTo(Collection<? super E> c, int maxElements);
}

```

总结归纳如下：
||抛出异常|返回特殊值|阻塞|超时|
|-|-|-|-|-|
|插入|add|offer|put|offer(e,timeout,unit)|
|移除|remove|poll|take|poll(time, unit)|
|检查|element|peek/contains|


<font color="red">*根据实际需求选取适合的插入、移除方法。</font>


实现BlockingQueue接口的实现类如下：</br></br>
![BlockingQueue及其子类](https://raw.githubusercontent.com/MuggleLee/PicGo/master/BlockingQueue%E5%8F%8A%E5%85%B6%E5%AD%90%E7%B1%BB.png)



ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列，遵循FIFO原则，可选择公平锁或者非公平锁，最大长度为Integer.MAX_VALUE。
LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列，遵循FIFO原则，默认和最大长度为Integer.MAX_VALUE。
LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。默认和最大长度为Integer.MAX_VALUE。
LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列，默认自然序进行排序，也可以自定义实现compareTo()方法来指定元素排序规则，不能保证同优先级元素的顺序。
DelayQueue：一个使用优先级队列实现的无界阻塞队列。
SynchronousQueue：不存储元素的阻塞队列，默认是非公平锁。


|队列|有界性|锁|数据结构|
|-|-|-|-|
|ArrayBlockingQueue|✔|✔|
|LinkedBlockingQueue|✔|✔|
|LinkedBlockingDeque|✔|✖|
|LinkedTransferQueue|✖|✔|
|PriorityBlockingQueue|✖|✔|
|DelayQueue|✖|✔|
|SynchronousQueue|✔|✔|


参考资料：
[http://www.cnblogs.com/zaizhoumo/p/7786793.html]()
