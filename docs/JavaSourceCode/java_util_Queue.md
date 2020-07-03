# JDK1.8 - Queue源码阅读

## 引言

一次面试中被问到了关于队列的问题，然后他问我队列是怎么实现的，我当时脱口而出的是LinkedList，然后就没有然后了，于是打算看看相关的源码。

## 要点总结

1. Queue继承了Collection接口，提供了insert，delete，inspect的操作，这些操作如果不能正常执行就会报一个exception。
2. Queue通常是以元素的进入顺序即FIFO(First in first out)排序的，这也是它有着先进先出的特征的关键所在了。像优先队列是实现了特有的比较方法才做到排序的。
3. offer(e) 与add(e)，offer在插入时如果不成功不会像add方法那样抛出一个异常。offer通常用在因为先天的限制而非错误的操作所导致的一些failure，比如说限长的队列。
4. remove()与poll()都是删除队尾的一个元素，他们的差别仅仅在于当queue为空时他们的反应是不一样的。remove是抛出异常，poll则返回一个null。
5. element()与peek()都是返回queue的头部。
6. Queue没有实现阻塞，如果你要使用阻塞的话，有一个BlockingQueue接口实现了阻塞。
7. Queue是不允许插入一个null的元素的，因为在queue为空时如果你进行poll()返回值也是null。但是他的一个实现了LinkedList没有遵守这个规则，不过我们应该避免这种null被插入queue的情形发生。
8. Queue的实现中通常不定义基于元素版本的equals方法和hashCode方法，而是从Object类继承基于identity的版本，因为对于具有相同元素但顺序不同的队列，并不总是定义了基于元素的相等性。

## 方法

### boolean add (E e)

概述：如果能够队列的空间足够并成功插入的情况下返回“true”，空间不足抛出IllegalStateException异常。

继承的是Collection的add方法；

抛出：

- IllegalStateException 空间不足；
- ClassCastException 欲插入的元素的类禁止插入队列；
- NullPointerException 空指针异常，元素为null；
- IllegalArgumentException 欲插入的元素的某些属性禁止插入队列；

### boolean offer(E e)

概述：如果能够队列的空间足够并成功插入的情况下返回“true”，空间不足不会抛出IllegalStateException异常，而是返回“false”。因此有容量限制的队列中该方法比add要好。

抛出：*(这里是没有空间不足的异常的)*

- ClassCastException 欲插入的元素的类禁止插入队列；

- NullPointerException 空指针异常，元素为null；
- IllegalArgumentException 欲插入的元素的某些属性禁止插入队列；

### E remove()

概述：移除队列的head元素并返回之。如果没有元素可以移除即队列为空，抛出NoSuchElementException。

抛出：

- NoSuchElementException 队列为空

### E poll()

概述：移除队列的head元素并返回之。如果没有元素可以移除即队列为空返回一个null。***这也是queue不建议插入null元素的原因所在了。***

无抛出。

### E element()

概述：取出队列的head元素并返回。如果没有元素可以返回即队列为空时抛出一个NoSuchElementException异常。

抛出：

- NoSuchElementException 队列为空；

### E peek()

概述：取出队列的head元素并返回。如果没有元素可以返回即队列为空时则返回null。***这也是queue不建议插入null元素的原因所在了。***

无抛出。

## 最后

近期我会坚持每天看看源码并整理出来，加油^__^!!!

[源码地址]([https://gitee.com/ozj127/experiment/blob/master/jdk1.8%E6%BA%90%E7%A0%81/java/util/Queue.java](https://gitee.com/ozj127/experiment/blob/master/jdk1.8源码/java/util/Queue.java))