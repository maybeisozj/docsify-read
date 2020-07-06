# volatile 关键字解读

## 概述

Volatile 变量具有 `synchronized` 的可见性特性，但是不具备原子特性。这就是说线程能够自动发现 volatile 变量的最新值。Volatile 变量可用于提供线程安全，但是只能应用于非常有限的一组用例：多个变量之间或者某个变量的当前值与修改后值之间没有约束。

volatile关键字可以说是synchronized关键字的轻量版，他保证了共享变量的可见性以及防止一定程度的指令重排。

### 特点

#### 保证volatile修饰的变量的可见性

可见性是指一个线程修改了一个共享变量(**多个线程可访问**)，其中一个修改了该变量的值，其他线程可以立即看到修改的值。

##### Java中存在的可见性问题

在Java内存模型规定所有的变量(**实例字段，静态字段和构成数组对象的元素，不包括局部变量和方法参数，因为它们是线程私有的**)都存储在主内存。每条线程还有自己的工作内存(**是每个线程私有的，类比CPU缓存**)，其中保存了被该线程使用的变量的主内存**副本**，线程对变量的所有操作都必须在工作内存中完成，不能直接对主内存的变量进行操作。**线程间变量值的传递需要通过主内存来完成**。

线程、工作内存、主内存工作交互如下图：

![image-20200701222923473](https://i.loli.net/2020/07/01/EeDXvQNTnRPKgd1.png)

图片来源：深入理解Java虚拟机第三版 12.3.1节。

正如上文所述，由于Java线程会拷贝主内存中的变量，他们的值不一定是相同。

#### 禁止对volatile修饰的变量重排序

##### Java中的重排序问题

比如单例模式的DCL(Double Checking Locking)，如下所示：

```java
public class sington{
    private static Sington instance;
    
    private Sington(){
        
    }
    
    public static Sington getInstance(){
        if (instance == null){	//DCL
            synchronized(Sington.class){
                if (instance == null){
                    instance = new Sington();
                }
            }
        }
        return instance;
    }
}
```

上面代码中的instance = new Sington()其底层会分为三个操作：

1. 分配一块内存；
2. 在内存上初始化成员变量；
3. 把instance引用指向内存；

在这三个操作中，2与3没有依赖关系可能会发生重排序问题，即可能会先把instance指向内存再初始化成员变量。此时另外一个线程拿到的是一个未完全初始化的对象，这是对其进行访问就会出错，这就是典型的"构造函数溢出"问题。

## 原理

### 可见性实现

　　由于线程本身并不直接与主内存进行数据的交互，而是通过线程的工作内存来完成相应的操作。这也是导致线程间数据不可见的本质原因。因此要实现volatile变量的可见性，直接从这方面入手即可。对volatile变量的写操作与普通变量的主要区别有两点：

　　（1）修改volatile变量时会强制将修改后的值刷新的主内存中。

　　（2）修改volatile变量后会导致其他线程工作内存中对应的变量值失效。因此，再读取该变量值的时候就需要重新从读取主内存中的值。

　　通过这两个操作，就可以解决volatile变量的可见性问题。

### 禁止重排序

volatile 关键字的底层原理是内存屏障实现的，在这之前，我们先了解以下as-if-serial和happen-before再介绍内存屏障。

#### as-if-serial语义

##### 单线程下的重排序规则

保证线程结果不会改变的前提下即操作之间没有数据依赖性的话，编译器和CPU可以任意重排序。

##### 多线程下的重排序规则

由于线程之间的依赖性太复杂，编译器和CPU没有办法完全理解这种依赖性并据此做出最合理的优化，因此编译器和CPU只保证单个线程的as-if-serial 语义。数据之间仍然相互影响，需要编译器和CPU的上层来确定。

#### happen-before

如果A happen-before B，那么A的执行结果必须对B可见，也就是保证跨线程的内存可见性。**A happen-before B 并不代表A发生在B之前，happen-before只确保如果A发生在B之前，那么A的执行结果必须对B可见。**

具体有：

- 同一个线程中的，前面的操作 happen-before 后续的操作。（即单线程内按代码顺序执行。但是，在不影响在单线程环境执行结果的前提下，编译器和处理器可以进行重排序，这是合法的。换句话说，这一是规则无法保证编译重排和指令重排）。
- 监视器上的解锁操作 happen-before 其后续的加锁操作。（Synchronized 规则）
- 对volatile变量的写操作 happen-before 后续的读操作。（volatile 规则）
- 线程的start() 方法 happen-before 该线程所有的后续操作。（线程启动规则）
- 线程所有的操作 happen-before 其他线程在该线程上调用 join 返回成功后的操作。
- 如果 a happen-before b，b happen-before c，则a happen-before c（传递性）。

第三条：volatile变量的保证有序性的规则。

#### 内存屏障

为了实现volatile可见性和happen-befor的语义。JVM底层是通过一个叫做“内存屏障”(Memory Barrier)的东西来实现的。内存屏障，也叫做内存栅栏，是一组处理器指令，用于实现对内存操作的顺序限制。

编译器的内存屏障是为了告诉编译器不要对指令进行重排。当编译完成之后这种内存屏障就消失了，CPU并不会感知到编译器中内存屏障的存在。

CPU的内存屏障是CPU提供的指令，可以由开发者显示调用。

JDK中的内存屏障：

从JDK8开始，在Unfase类中提供了三个内存屏障函数，如下:

```java
public native void loadFence();
public native void storeFence();
public native void fullFence();
```

在理论层面可以分为以下四种：

（1）LoadLoad 屏障
执行顺序：Load1—>Loadload—>Load2
确保Load2及后续Load指令加载数据之前能访问到Load1加载的数据。

（2）StoreStore 屏障
执行顺序：Store1—>StoreStore—>Store2
确保Store2以及后续Store指令执行前，Store1操作的数据对其它处理器可见。

（3）LoadStore 屏障
执行顺序： Load1—>LoadStore—>Store2
确保Store2和后续Store指令执行前，可以访问到Load1加载的数据。

（4）StoreLoad 屏障
执行顺序: Store1—> StoreLoad—>Load2
确保Load2和后续的Load指令读取之前，Store1的数据对其他处理器是可见的。

两者的对应关系如下：

- loadFence=LoadLoad + LoadStore;
- storeFence=StoreStore+LoadStore;
- fullFence=loadFence+storeFence+StoreLoad;

#### 实现volatile语义的一种参考做法：

1. 在volatile写操作的前面插入一个StoreStore屏障。保证volatile写操作不会和之前的写操作重排序；
2. 在volatile写操作的后面插入一个StoreLoad屏障。保证volatile写操作不会和之后的读操作重排序；
3. 在volatile读操作的后面插入一个LoadLoad屏障+LoadStore屏障。保证volatile读操作不会和之后的读操作、写操作重排序。

## 使用

### 正确使用 volatile 变量的条件

我们只能在有限的一些情形下使用 volatile 变量替代锁。要使 volatile 变量提供理想的线程安全，必须同时满足下面两个条件：

- 对变量的写操作不依赖于当前值。
- 该变量没有包含在具有其他变量的不变式中。

### 正确使用 volatile 的模式

很多并发性专家事实上往往引导用户远离 volatile 变量，因为使用它们要比使用锁更加容易出错。然而，如果谨慎地遵循一些良好定义的模式，就能够在很多场合内安全地使用 volatile 变量。要始终牢记使用 volatile 的限制 —— 只有在状态真正独立于程序内其他内容时才能使用 volatile —— 这条规则能够避免将这些模式扩展到不安全的用例。

#### 模式1：状态标志

也许实现 volatile 变量的规范使用仅仅是使用一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或请求停机。

很多应用程序包含了一种控制结构，形式为 “在还没有准备好停止程序时再执行一些工作”，如清单 2 所示：

```java
    //  将 volatile 变量作为状态标志使用
    volatile boolean shutdownRequested;

    ...

    public void shutdown() { shutdownRequested = true; }

    public void doWork() { 
        while (!shutdownRequested) { 
            // do stuff
        }
    }
```

很可能会从循环外部调用 `shutdown()` 方法 —— 即在另一个线程中 —— 因此，需要执行某种同步来确保正确实现 `shutdownRequested` 变量的可见性。（可能会从 JMX 侦听程序、GUI 事件线程中的操作侦听程序、通过 RMI 、通过一个 Web 服务等调用）。然而，使用 `synchronized` 块编写循环要比使用清单 2 所示的 volatile 状态标志编写麻烦很多。由于 volatile 简化了编码，并且状态标志并不依赖于程序内任何其他状态，因此此处非常适合使用 volatile。

这种类型的状态标记的一个公共特性是：通常只有一种状态转换；`shutdownRequested` 标志从 `false` 转换为 `true`，然后程序停止。这种模式可以扩展到来回转换的状态标志，但是只有在转换周期不被察觉的情况下才能扩展（从 `false` 到 `true`，再转换到 `false`）。此外，还需要某些原子状态转换机制，例如原子变量。

#### 模式2：一次性安全发布（one-time safe publication）

缺乏同步会导致无法实现可见性，这使得确定何时写入对象引用而不是原语值变得更加困难。在缺乏同步的情况下，可能会遇到某个对象引用的更新值（由另一个线程写入）和该对象状态的旧值同时存在。（这就是造成著名的双重检查锁定（double-checked-locking）问题的根源，其中对象引用在没有同步的情况下进行读操作，产生的问题是您可能会看到一个更新的引用，但是仍然会通过该引用看到不完全构造的对象）。

实现安全发布对象的一种技术就是将对象引用定义为 volatile 类型。清单 3 展示了一个示例，其中后台线程在启动阶段从数据库加载一些数据。其他代码在能够利用这些数据时，在使用之前将检查这些数据是否曾经发布过。

```java
    // 将 volatile 变量用于一次性安全发布
    public class BackgroundFloobleLoader {
        public volatile Flooble theFlooble;

        public void initInBackground() {
            // do lots of stuff
            theFlooble = new Flooble();  // this is the only write to theFlooble
        }
    }

    public class SomeOtherClass {
        public void doWork() {
            while (true) { 
                // do some stuff...
                // use the Flooble, but only if it is ready
                if (floobleLoader.theFlooble != null) 
                    doSomething(floobleLoader.theFlooble);
            }
        }
    }
```

如果 `theFlooble` 引用不是 volatile 类型，`doWork()` 中的代码在解除对 `theFlooble` 的引用时，将会得到一个不完全构造的 `Flooble`。

该模式的一个必要条件是：被发布的对象必须是线程安全的，或者是有效的不可变对象（有效不可变意味着对象的状态在发布之后永远不会被修改）。volatile 类型的引用可以确保对象的发布形式的可见性，但是如果对象的状态在发布后将发生更改，那么就需要额外的同步。

#### 模式3：独立观察（independent observation）

安全使用 volatile 的另一种简单模式是：定期 “发布” 观察结果供程序内部使用。例如，假设有一种环境传感器能够感觉环境温度。一个后台线程可能会每隔几秒读取一次该传感器，并更新包含当前文档的 volatile 变量。然后，其他线程可以读取这个变量，从而随时能够看到最新的温度值。

使用该模式的另一种应用程序就是收集程序的统计信息。清单 4 展示了身份验证机制如何记忆最近一次登录的用户的名字。将反复使用 `lastUser` 引用来发布值，以供程序的其他部分使用。

```java
//将 volatile 变量用于多个独立观察结果的发布
public class UserManager {
    public volatile String lastUser;
    public boolean authenticate(String user, String password) {
        boolean valid = passwordIsValid(user, password);
        if (valid) {
        	User u = new User();
            activeUsers.add(u);
            lastUser = user;
        }
        return valid;
    }
}
```

该模式是前面模式的扩展；将某个值发布以在程序内的其他地方使用，但是与一次性事件的发布不同，这是一系列独立事件。这个模式要求被发布的值是有效不可变的 —— 即值的状态在发布后不会更改。使用该值的代码需要清楚该值可能随时发生变化。

#### 模式4：“volatile bean” 模式

volatile bean 模式适用于将 JavaBeans 作为“荣誉结构”使用的框架。在 volatile bean 模式中，JavaBean 被用作一组具有 getter 和/或 setter 方法 的独立属性的容器。volatile bean 模式的基本原理是：很多框架为易变数据的持有者（例如 `HttpSession`）提供了容器，但是放入这些容器中的对象必须是线程安全的。

在 volatile bean 模式中，JavaBean 的所有数据成员都是 volatile 类型的，并且 getter 和 setter 方法必须非常普通 —— 除了获取或设置相应的属性外，不能包含任何逻辑。此外，对于对象引用的数据成员，引用的对象必须是有效不可变的。（这将禁止具有数组值的属性，因为当数组引用被声明为 `volatile` 时，只有引用而不是数组本身具有 volatile 语义）。对于任何 volatile 变量，不变式或约束都不能包含 JavaBean 属性。清单 5 中的示例展示了遵守 volatile bean 模式的 JavaBean：

```java
// 遵守 volatile bean 模式的 Person 对象
@ThreadSafe
public class Person {
    private volatile String firstName;
    private volatile String lastName;
    private volatile int age;
 
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
 
    public void setFirstName(String firstName) { 
        this.firstName = firstName;
    }
 
    public void setLastName(String lastName) { 
        this.lastName = lastName;
    }
 
    public void setAge(int age) { 
        this.age = age;
    }
}
```

#### 模式5：开销较低的读－写锁策略

目前为止，您应该了解了 volatile 的功能还不足以实现计数器。因为 `++x` 实际上是三种操作（读、添加、存储）的简单组合，如果多个线程凑巧试图同时对 volatile 计数器执行增量操作，那么它的更新值有可能会丢失。

然而，如果读操作远远超过写操作，您可以结合使用内部锁和 volatile 变量来减少公共代码路径的开销。清单 6 中显示的线程安全的计数器使用 `synchronized` 确保增量操作是原子的，并使用 `volatile` 保证当前结果的可见性。如果更新不频繁的话，该方法可实现更好的性能，因为读路径的开销仅仅涉及 volatile 读操作，这通常要优于一个无竞争的锁获取的开销。

```java
// 结合使用 volatile 和 synchronized 实现 “开销较低的读－写锁”
@ThreadSafe
public class CheesyCounter {
    // Employs the cheap read-write lock trick
    // All mutative operations MUST be done with the 'this' lock held
    @GuardedBy("this") private volatile int value;
 
    public int getValue() { return value; }
 
    public synchronized int increment() {
        return value++;
    }
}
```

之所以将这种技术称之为 “开销较低的读－写锁” 是因为您使用了不同的同步机制进行读写操作。因为本例中的写操作违反了使用 volatile 的第一个条件，因此不能使用 volatile 安全地实现计数器 —— 您必须使用锁。然而，您可以在读操作中使用 volatile 确保当前值的*可见性*，因此可以使用锁进行所有变化的操作，使用 volatile 进行只读操作。其中，锁一次只允许一个线程访问值，volatile 允许多个线程执行读操作，因此当使用 volatile 保证读代码路径时，要比使用锁执行全部代码路径获得更高的共享度 —— 就像读－写操作一样。然而，要随时牢记这种模式的弱点：如果超越了该模式的最基本应用，结合这两个竞争的同步机制将变得非常困难。

## 参考

[[Java 并发编程：volatile的使用及其原理](https://www.cnblogs.com/paddix/p/5428507.html)](https://www.cnblogs.com/paddix/p/5428507.html)

[正确使用 Volatile 变量](https://www.ibm.com/developerworks/cn/java/j-jtp06197.html)