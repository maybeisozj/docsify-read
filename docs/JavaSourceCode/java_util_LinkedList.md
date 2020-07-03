

# JDK1.8-LinkedList源码阅读

## 引言

LinkedList我们通常都是用它用实现我们的Queue的，可能只有我？现在我们一起看看它的源码如何。

## 总结

1. 实现了List以及Deque(双端队列)的接口，元素节点保存有pre以及next，这意味着他可以从前面开始插入(遍历)，也可以从后面插入(遍历)。这一点在它的方法里就有体现。

2. 如我们在Queue那里看见的一样，LinkedList并没有阻止null元素的进入，严格来说这是不符合Queue的约定的。

3. LinkedList是线程不安全的，多个线程访问时如果有结构性变化（add，delete）需要实现同步。这通常是通过对一些自然封装列表的对象进行同步来实现的。如果没有，可以在创建时使用保证同步：

   ```java
   List list = Collections.synchronizedList(new LinkedList(...));
   ```

4. LinkedList利用迭代器遍历。在使用迭代器时如果使用非迭代器提供的add以及remove方法，迭代器会抛出ConcurrentModificationException异常，对于并发修改保护数据有着独到之处。

## 变量

```java
    /**
    * 实际元素个数
    */
 	transient int size = 0;

    /**
     * 第一个节点的指针
     */
    transient Node<E> first;

    /**
     * 最后一个节点的指针
     */
    transient Node<E> last;

	/**
	* 内部节点类，正因为维护了前后节点的
	* 指针才可以双向遍历
	*/
	private static class Node<E> {
        E item;//节点值
        Node<E> next;//下一个
        Node<E> prev;//上一个

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

## 方法

### 构造方法

提供了两个构造方法，一个是一个空的构造，一个可以在初始化的时候传入Collection集合的一个对象。

```java
/**
 * 构造空的list
 */
public LinkedList() {
}

/**
 * 可以传入Collection进行初始化，顺序与元素在迭代器的顺序有关
 */
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

### 内部方法

void linkFirst(E e)

概述：插入到第一个元素之前，size++，如果为空则 first = last = newNode

void linkLast(E e)

概述：插入到最后一个元素之后，size++，如果为空则 first = last = newNode

void linkBefore(E e, Node<E> succ)

概述：插入到指定节点元素之前，如果succ.pre == null 则可以断定succ为第一个节点，此时等价于linkFirst方法

E unlinkFirst(Node<E> f)

概述：从链表中断开节点f并返回该节点值，即删除该节点及其之前的节点，此时first=f.next，并且利用f.item = null，f.pre = null，f.next=null 加快垃圾回收

E unlinkLast(Node<E> f)

概述：从链表中断开节点f并返回该节点值，即删除该节点及其以后的节点，此时last=f.pre，并且利用f.item = null，f.pre = null，f.next=null 加快垃圾回收

E unlink(Node<E> f)

概述：从链表中断开节点f并返回该节点值，即删除该节点，此时将该节点的pre指向next，并且利用f.item = null，f.pre = null，f.next=null 加快垃圾回收

### 外部方法

E getFirst()

概述：取得第一个节点的值，如果此时为空抛出 NoSuchElementException 异常

E getLast()

概述：取得最后一个节点的值，如果此时为空抛出 NoSuchElementException 异常

E removeFirst()

概述：删除第一个节点并返回该节点值，如果此时为空抛出 NoSuchElementException 异常

E removeLast()

概述：删除最后一个节点并返回该节点值，如果此时为空抛出 NoSuchElementException 异常

void addFirst(E e)

概述：调用linkFirst方法，在开始处添加一个节点

void addLast(E e)

概述：调用linkLast方法，在末尾处添加一个节点

boolean add(E e)

概述：调用linkLast方法在末尾处添加一个节点，返回true

void add(int index, E element)

概述：在指定位置之前加入新节点，值为element，需要判断是否跃界

boolean addAll(Collection<? extends E> c)

概述：调用addAll(size,c)方法在末尾处添加所有值

boolean addAll(int index, Collection<? extends E> c)

概述：指定index位置处开始添加所有值，这里需要检验index是否跃界以及c是否为空指针

void clear()

概述：清空链表，进行x.item = null 等操作加快垃圾回收

int size()

概述：返回链表大小

boolean contains(Object o)

概述：如果list里存在该o则返回它第一次出现的索引，否则返回-1，调用indexOf方法

int indexOf(Object o)

概述：存在该o则返回它第一次出现的索引，否则返回-1，这里的o可以为null

boolean remove(Object o)

概述：如果存在这个元素则删除第一次出现的节点，否则返回false

void remove(int index)

概述：删除指定位置的节点，判断index是否跃界

E get(int index)

概述：取得指定位置的节点值，需要判断index是否跃界，无跃界调用node(index)取得节点

E set(int index, E element)

概述：设置指定位置的节点的值为element，需要判断index是否跃界

boolean isElementIndex(int index)

概述：判断index是否在链表范围之内，即>= 0&& <=size

boolean isPositionIndex(int index)

概述：判断index是否在链表范围之内，即>= 0&& <=size

Node<E> node(int index)

概述：取得指定位置的节点，这里写的时候判断了一下index接近0还是size以决定从哪一方开始

int indexOf(Object o)

概述：返回对象o的第一次出现的index，不存在返回null

int lastIndexOf(Object o)

概述：返回对象o的最后一次出现的index，不存在返回null