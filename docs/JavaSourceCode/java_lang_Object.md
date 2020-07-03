# java.lang.Object 阅读总结

## 概要

Object类是所有类的父类，你不用显式的用extend是继承他，如果没有写默认就是继承Object类。

## 方法

```java
public final native Class<?> getClass()
public native int hashCode()
public boolean equals(Object obj)
protected native Object clone() throws CloneNotSupportedException
public String toString()
public final native void notify()
public final native void notifyAll()
public final native void wait(long timeout) throws InterruptedException
public final void wait(long timeout, int nanos) throws InterruptedException
public final void wait() throws InterruptedException
protected void finalize() throws Throwable { }
```

### getClass方法

修饰符为**final**，以及**native**，即不可以重写以及这个方法是调用本地方法(**C/C++实现**)实现的。

返回的是代表此对象的运行时类的{@code Class}对象。

调用方式

```java
ObjectTest ot = new ObjectTest();
Class<? extends ObjectTest> class1 = ot.getClass();
out.println(class1);
```

### hashCode 方法

修饰符为**native**，没有final是因为推荐编写类时重写**hasCode和equals**方法。

返回值为一个散列值(hash code value)，类型是整型，这个值是在HashTable或者HashMap中使用的，

哈希码的通用约定如下：

1. 在java程序执行过程中，在一个对象没有被改变(**这里改变说的是用在equals的属性发生了变化**)的前提下，无论这个对象被调用多少次，hashCode方法都会返回相同的整数值。对象的哈希码没有必要在不同的程序中保持相同的值。
2. 如果2个对象使用equals方法进行比较并且相同的话，那么这2个对象的hashCode方法的值也必须相等。
3. 如果根据equals方法，得到两个对象不相等，那么这2个对象的hashCode值不需要必须不相同。但是，不相等的对象的hashCode值不同的话可以提高哈希表的性能。(**即减少了冲突**)

通常情况下，不同的对象产生的哈希码是不同的。默认情况下，对象的哈希码是通过将该对象的内部地址转换成一个整数来实现的。

### equals 方法

这个方法实现的是与另一个对象实例的比较，如果相同返回true，不同返回false。这个方法也是推荐大家重写的，Object的实现如下：

```java
	public boolean equals(Object obj) {
        return (this == obj);
    }
```

它比较的是地址。

equals方法在非空对象引用上的特性：

1. reflexive，自反性。任何非空引用值x，对于x.equals(x)必须返回true
2. symmetric，对称性。任何非空引用值x和y，如果x.equals(y)为true，那么y.equals(x)也必须为true
3. transitive，传递性。任何非空引用值x、y和z，如果x.equals(y)为true并且y.equals(z)为true，那么x.equals(z)也必定为true
4. consistent，一致性。任何非空引用值x和y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改
5. 对于任何非空引用值 x，x.equals(null) 都应返回 false

#### 关于== 与equals

== 比较的是基础类型如int，string，char是比较的是他们的值，其他对象时是地址。

equals 则是基于内部规则，没有重写则是比较地址，如String则重写了equals方法，比较的是值。

### clone 方法

修饰符是native，本地方法。

返回值是一个该对象的一个复制品，一般的有 x.clone() != x && x.clone().getClass() == x.getClass() 为true。

注意事项：

1. 深拷贝还是浅拷贝要看我们自己的实现，实现CloneAble接口并重写clone方法达到我们的目的；
2. 没有实现CloneAble接口的会抛出CloneNotSupportedException 异常；
3. Object没有实现CloneAble接口，所以调用会抛出异常；

#### 深拷贝or浅拷贝

对于基础类型，深拷贝与浅拷贝表现是一样的，都是复制他的值，并不会影响之前的值。

对于引用类型，深拷贝相当于new 一个新的实例，浅拷贝则是复制一个指向该对象实例的地址而已。因此，深拷贝得到的新对象的修改不会影响原来的对象实例，而浅拷贝则会。(**上是浅拷贝，下是深拷贝**)

![img](https://i.loli.net/2020/06/25/E1nFjHQuwNSgK2k.png)

![img](https://i.loli.net/2020/06/25/p2j3LBwCs7yEPzW.png)

### toString 方法

Object对象的默认实现，即输出类的名字@实例的哈希码的16进制(**class 的名称 + @ + 地址**)：

```Java
getClass().getName() + "@" + Integer.toHexString(hashCode());
```

在out.println()方法中参数为实例对象时如会自动调用对象的toString()方法。

### notify 方法

唤醒一个等待该对象实例的线程(**该线程通过wait方法等待**)。如果有多个则选择一个唤醒。

**唤醒的线程需要等到当前线程释放该对象实例的锁才可以正常运行**

notify方法只能被作为此对象监视器的所有者的线程来调用。一个线程要想成为对象监视器的所有者，可以使用以下3种方法：

1. 执行对象的同步实例方法
2. 使用synchronized内置锁
3. 对于Class类型的对象，执行同步静态方法

一次只能有一个线程拥有对象的监视器。

**如果当前线程不是此对象监视器的所有者的话会抛出IllegalMonitorStateException异常。**

### notifyAll方法

跟notify一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。

同样，如果当前线程不是对象监视器的所有者，那么调用notifyAll同样会发生IllegalMonitorStateException异常。

### wait(long timeout)方法

wait(long timeout)方法同样是一个native方法，并且也是final的，不允许子类重写。

wait方法会让当前线程等待直到另外一个线程调用对象的notify或notifyAll方法，或者超过参数设置的timeout超时时间。

**跟notify和notifyAll方法一样，当前线程必须是此对象的监视器所有者，否则还是会发生IllegalMonitorStateException异常。**

wait方法会让当前线程(我们先叫做线程T)将其自身放置在对象的等待集中，并且放弃该对象上的所有同步要求。出于线程调度目的，线程T是不可用并处于休眠状态，直到发生以下四件事中的任意一件：

1. 其他某个线程调用此对象的notify方法，并且线程T碰巧被任选为被唤醒的线程
2. 其他某个线程调用此对象的notifyAll方法
3. 其他某个线程调用Thread.interrupt方法中断线程T
4. 时间到了参数设置的超时时间。如果timeout参数为0，则不会超时，会一直进行等待

所以可以理解wait方法相当于放弃了当前线程对对象监视器的所有者(也就是说释放了对象的锁)

之后，线程T会被等待集中被移除，并且重新进行线程调度。然后，该线程以常规方式与其他线程竞争，以获得在该对象上同步的权利；一旦获得对该对象的控制权，该对象上的所有其同步声明都将被恢复到以前的状态，这就是调用wait方法时的情况。然后，线程T从wait方法的调用中返回。所以，从wait方法返回时，该对象和线程T的同步状态与调用wait方法时的情况完全相同。

在没有被通知、中断或超时的情况下，线程还可以唤醒一个所谓的虚假唤醒 (spurious wakeup)。虽然这种情况在实践中很少发生，但是应用程序必须通过以下方式防止其发生，即对应该导致该线程被提醒的条件进行测试，如果不满足该条件，则继续等待。换句话说，等待应总是发生在循环中，如下面的示例：

```
synchronized (obj) {
    while (<condition does not hold>)
        obj.wait(timeout);
        ... // Perform action appropriate to condition
}
```

如果当前线程在等待之前或在等待时被任何线程中断，则会抛出InterruptedException异常。在按上述形式恢复此对象的锁定状态时才会抛出此异常。

### wait(long timeout, int nanos) 方法

跟wait(long timeout)方法类似，多了一个nanos参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上nanos毫秒。

需要注意的是 wait(0, 0)和wait(0)效果是一样的，即一直等待。

### wait 方法

跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念。

以下这段代码直接调用wait方法会发生IllegalMonitorStateException异常，这是因为调用wait方法需要当前线程是对象监视器的所有者：

```
Factory factory = new Factory();
factory.wait();
```

一般情况下，wait方法和notify方法会一起使用的，wait方法阻塞当前线程，notify方法唤醒当前线程。

### finalize方法

finalize方法是一个protected方法，Object类的默认实现是不进行任何操作。

该方法的作用是实例被垃圾回收器回收的时候触发的操作，就好比 “死前的最后一波挣扎”。

**以下内容来自深入理解Java虚拟机:JVM高级特性第三版3.2节**

在可达性分析算法中判定为不可达的对象,也不是"非死不可"的,这时候它们暂时还处于“缓刑"阶段,要真正宣告一个对象死亡,至少要经历两次标记过程:

如果对象在进行可达性分析后发现没有与GC Roots相连接的引用链,那它将会被第一次标记,随后进行一次筛选,筛选的条件是此对象是否有必要执行finalize()方法。

假如对象没有覆盖finalize()方法,或者finalize()方法已经被虚拟机调用过,那么虚拟机将这两种情况都视为"没有必要执行"如果这个对象被判定为确有必要执行finalize()方法,那么该对象将会被放置在一个名为**F-Queue的队列**之中,并在稍后由一条由**虚拟机自动建立的、低调度优先级**的Finalizer线程去执行它们的finalize()方法。

这里所说的“执行"是指虚拟机会触发这个方法开始运行,但**并不承诺一定会等待它运行结束**。这样做的原因是,如果某个对象的finalize()方法执行缓慢,或者更极端地发生了死循环,将很可能导致F-Queue队列中的其他对象永久处于等待,甚至导致整个内存回收子系统的崩溃。finalize()方法是对象逃脱死亡命运的最后一次机会,稍后收集器将对F-Queue中的对象进行第二次小规模的标记,如果对象要在finalize()中成功拯救自己-只要**重新与引用链上的任何一个对象建立关联**即可,譬如把自己(this关键字)赋值给某个类变量或者对象的成员变量,那在第二次标记时它将被移出"即将回收"的集合;如果对象这时候还没有逃脱,那基本上它就真的要被回收了。

```java
/**
* 此代码演示了两点:
* 1.对象可以在被GC时自我拯救。
* 2.这种自救的机会只有一次,因为一个对象的finalize()方法最多只会被系统自动调用一次
**/
public class FinalizeEscapeGC{
    public static FinalizeEscapeGC SAVE HOOK = nul;
    public void isAlive() { 
    	System.out.printin("yes, i am still alive :y");
    }
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.printin("finalize method executed!");
        EinalizeEscapeGC.SAVE HOOK = this;
    }
    
    public static void main(StringIl args) throws Throwable { 
        SAVE HOOK =new FinalizeEscapeGC();//对象第一次成功拯救自己
        SAVE HOOK =null:System.gc();//因为Finalizer方法优先级很低,暂停0.5秒,以等待它
        Thread.sleep(500);
        if (SAVE HOOK!= null) { 
            SAVE HOOK.isAlive();
        } else {
            System.out.printin("no, i am dead:(");
            //下面这段代码与上面的完全相同,但是这次自救却失败了
            SAVE HOOK = null;
            System.gc();//因为Finalzer方法优先级很低,暂停0.5秒,以等待它
            Thread.sleep(500);
            if (SAVE HOOK != null) { 
                SAVE HOOK.isAlive();
            } else {
                System.out.println("no, I am dead.");
            }
        }
    }
```