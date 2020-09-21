# final关键字

## 概述

final可能是我们遇见的较多的关键字了，当然，这里的意思是与volatile以及synchronized关键字做比较的。它的意思简单的理解就是**"不可再修改"**。它可以用来修饰变量，方法以及类，比如String就是一个用final修饰的类。

## 修饰变量

### 类成员变量

final变量有成员变量或者是本地变量(方法内的局部变量)，在类成员中final经常和static一起使用，作为类常量使用。**其中类常量必须在声明时初始化，final成员常量可以在构造函数初始化。**

如下：

```java
package com.java.test.finalTest;

public class FinalTest {

    private final String a;// 这里不进行初始化的话可以到构造函数进行初始化

    private static final String b; // 这里会报错，因为常量在常量池中就存在了，调用时不需要类的初始化，所以必须在声明时初始化

    FinalTest(){
        a = "a";
    }
}

```

- 类常量必须进行在静态初始化块中指定初始值或者声明该类变量时指定初始值，而且只能在这两个地方之一进行指定；

- ```java 
  package com.java.test.finalTest;
  
  public class FinalTest {
  
      // 初始化位置1
      private static final String a = "a";
  
      private static final String b;
  
      // 初始化位置2
      static {
          b = "b";
      }
  }
  ```

- 实例变量必须在非静态初始化块，声明该实例变量或者在构造器中指定初始值，而且只能在这三个地方进行指定;

- ```java
  package com.java.test.finalTest;
  
  public class FinalTest {
  
      private final String a;
  
      private final String b = "b";
      
      private final String c;
  
      {
          a = "a";
      }
      
      public FinalTest(){
          c = "c";
      }
  }
  ```

### 局部变量

#### 基础数据类型

final局部变量由程序员进行显式初始化， 如果final局部变量已经进行了初始化则后面就不能再次进行更改， 如果final变量未进行初始化，可以进行赋值，当且仅有一次赋值，一旦赋值之后再次赋值就会出错。如下：

```java
	private void testOne(){
        final int a = 1;
        a = 2; // 这里报错，因为已经进行初始化了，不能进行修改
        
        final int b;
        b = 1;// 可以赋值
        b = 2;// 报错，因为已经进行赋值操作了即已经初始化了
    }
```

final修饰基本数据类型变量时，不能对基本数据类型变量重新赋值， 因此基本数据类型变量不能被改变。

#### 引用数据类型

基础数据类型无法进行修改，那么引用数据类型呢？为了方便测试，我们加入一个内部类Person，他有姓名和年龄，测试代码如下:

```java
package com.java.test.finalTest;

public class FinalTest {

    public static void main(String[] args) {
        testTwo();
    }

    private static void testTwo(){
        final Person person = new Person("hhh", 18);
        System.out.println(person);
        // person = new Person("xxx", 18);
        person.setAge(19);
        System.out.println(person);
    }

    static class Person {
        String name ;
        int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        @Override
        public String toString() {
            return "Person{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}

----------------输出结果------------------------
Person{name='hhh', age=18}
Person{name='hhh', age=19}
```

可以看见，person的年龄被我们修改了，这是为什么啊？不是说final是不可变的嘛？那么他怎么就修改成功了呢？

其实final修饰引用类型变量时，它仅仅保存的是一个引用，final只保证这个引用类型变量所引用的地址不会发生改变， 即一直引用这个对象，但这个对象属性是可以改变的。

## 修饰方法

将方法声明为final，在编译的时候就已经静态绑定了，不需要在运行时动态绑定。final方法调用时使用的是invokespecial指令。被final修饰的方法不可以进行重写，但是可以进行重载。如下：

```java
package com.java.test.finalTest;

public class FinalTestTwo {

    class Person {
        private String name ;
        
        private int age;

        public final void print(){
            System.out.println("print");
        }
        
        // 可以
        public final void print(int i){
            System.out.println("print2");
        }
    }

    class Student extends Person{

        private int sex;

        // 报错
        @Override
        public final void print(){
            System.out.println("override");
        }
    }
}

```

## 修饰类

当一个类被final修饰时，该类是**不能被子类继承**的。 子类继承往往可以重写父类的方法和改变父类属性，会带来一定的安全隐患， 因此，当一个类不希望被继承时就可以使用final修饰。

final经常会被用作不变类上。我们先来看看什么是不可变类：

- 使用private和final修饰符来修饰该类的成员变量
- 提供带参的构造器用于初始化类的成员变量
- 仅为该类的成员变量提供getter方法，不提供setter方法，因为普通方法无法修改fina修饰的成员变量
- 如果有必要就重写Object类的hashCode()和equals()方法，应该保证用equals()判断相同的两个对象其Hashcode值也是相等的。

JDK中提供的八个包装类和String类都是不可变类。

## final域重排序规则

### final为基本类型

```java
public class FinalDemo {    private int a;  //普通域
    private final int b; //final域-->int基本类型
    private static FinalDemo finalDemo;//引用类型，但不是final修饰的

    public FinalDemo() {
        a = 1; // 1. 写普通域
        b = 2; // 2. 写final域
    }    
    public static void writer() {
        finalDemo = new FinalDemo();
    }    
    public static void reader() {        
    	FinalDemo demo = finalDemo; // 3.读对象引用
        int a = demo.a;    //4.读普通域
        int b = demo.b;    //5.读final域
    }
}
```

假设线程A在执行writer()方法，线程B执行reader()方法。

##### 写final域重排序规则

写final域的重排序规则----禁止对final域的写重排序到构造函数之外，这个规则的实现主要包含了两个方面：

- JMM禁止编译器把final域的写重排序到构造函数之外
- 编译器会在final域写之后，构造函数return之前，插入一个StoreStore屏障。 这个屏障可以禁止处理器把final域的写重排序到构造函数之外。 (参见 StoreStore Barriers的说明：在Store1;Store2之间插入StoreStore，确保Store1对 其他处理器可见(刷新内存)先于Store2及所有后续存储指令的存储)

writer方法中，实际上做了两件事：

- 构造了一个FinalDemo对象
- 把这个对象赋值给成员变量finalDemo

可能的执行时序图如下：

![](https://raw.githubusercontent.com/maybeisozj/images/master/20200706175557.png)

a,b之间没有数据依赖性，普通域（普通变量）a可能会被重排序到构造函数之外， 线程B就有可能读到的是普通变量a初始化之前的值（零值），这样就可能出现错误。

final域变量b，根据重排序规则，会禁止final修饰的变量b重排序到构造函数之外，从而b能够正确赋值， 线程B就能够读到final变量初始化后的值。

因此，写final域的重排序规则可以确保：在对象引用为任意线程可见之前，对象的final域已经被正确初始化过了。 普通域不具有这个保障，比如在上例，线程B有可能就是一个未正确初始化的对象finalDemo。

##### 读final域重排序规则

读final域重排序规则:在一个线程中，初次读对象引用和初次读该对象包含的final域，JMM会禁止这两个操作的重排序。 （注意，这个规则仅仅是针对处理器）， 处理器会在读final域操作的前面插入一个LoadLoad屏障。 实际上，读对象的引用和读该对象的final域存在间接依赖性，一般处理器不会重排序这两个操作。 但是有一些处理器会重排序，因此，这条禁止重排序规则就是针对这些处理器而设定的。

read方法主要包含了三个操作：

- 初次读引用变量finalDemo
- 初次读引用变量finalDemo的普通域a
- 初次读引用变量finalDemo的final域b

假设线程A写过程没有重排序，那么线程A和线程B有一种的可能执行时序如下：

![](https://raw.githubusercontent.com/maybeisozj/images/master/20200706175532.png)

读对象的普通域被重排序到了读对象引用的前面就会出现线程B还未读到对象引用就在读取该对象的普通域变量，这显然是错误的操作。

final域的读操作就“限定”了在读final域变量前已经读到了该对象的引用，从而就可以避免这种情况。

因此，读final域的重排序规则可以确保：在读一个对象的final域之前，一定会先读这个包含这个final域的对象的引用。

### final为引用类型

```java
public class FinalReferenceDemo {    
	final int[] arrays; //arrays是引用类型
    
    private FinalReferenceDemo finalReferenceDemo;    public FinalReferenceDemo() {
        arrays = new int[1];  //1 
        arrays[0] = 1;        //2
    }    
    
    public void writerOne() {
        finalReferenceDemo = new FinalReferenceDemo(); //3
    }    
    
    public void writerTwo() {
        arrays[0] = 2;  //4
    }    
    
    public void reader() {        
        if (finalReferenceDemo != null) {  //5
            int temp = finalReferenceDemo.arrays[0];  //6
        }
    }
}
```

##### 对final修饰的对象的成员域进行写操作

针对引用数据类型，final域写针对编译器和处理器重排序增加了这样的约束： 在构造函数内对一个final修饰的对象的成员域的写入，与随后在构造函数之外把这个被构造的对象的引用赋给一个引用变量，这两个操作是不能被重排序的。 注意这里的是“增加”也就说前面对final基本数据类型的重排序规则在这里还是使用。

线程线程A执行wirterOne方法，执行完后线程B执行writerTwo方法，线程C执行reader方法。 下图就以这种执行时序出现的一种情况来讨论：

![](https://raw.githubusercontent.com/maybeisozj/images/master/20200706175625.png)

对final域的写禁止重排序到构造方法外，因此1和3不能被重排序。 由于一个final域的引用对象的成员域写入不能与在构造函数之外将这个被构造出来的对象赋给引用变量重排序， 因此2和3不能重排序。

##### 对final修饰的对象的成员域进行读操作

JMM可以确保线程C至少能看到写线程A对final引用的对象的成员域的写入，即能看到arrays[0] = 1，而 写线程B对数组元素的写入可能看到可能看不到。 JMM不保证线程B的写入对线程C可见，线程B和线程C之间存在数据竞争，此时的结果是不可预知的。 如果想要可见，可使用锁或者volatile。

### final重排序的总结

|              | final写                                                      | final域读                                                    |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 基本数据类型 | 禁止final域写与构造方法重排序，即禁止final域写重排序到构造方法之外，从而保证该对象对所有线程可见时，该对象的final域全部已经初始化过 | 禁止初次读对象的引用与读该对象包含的final域的重排序，保证了在读一个对象的final域之前，一定会先读这个包含这个final域的对象的引用 |
| 引用数据类型 | 额外增加约束：构造函数内对一个final修饰的对象的成员域的写入，与随后在构造函数之外把这个被构造的对象的引用赋给一个引用变量，这两个操作是不能被重排序的 |                                                              |

## 原理

借助内存屏障。

写final域会要求编译器在final域写之后，构造函数返回前插入一个StoreStore屏障。

读final域的重排序规则会要求编译器在读final域的操作前插入一个LoadLoad屏障。

## final 与 static

final 和 static 在一起使用就会发生神奇的化学反应，他们同时使用时即可修饰成员变量，也可修饰成员方法。

对于成员变量，该变量一旦赋值就不能改变，我们称它为“全局常量”。可以通过类名直接访问。

对于成员方法，则是不可继承和改变。可以通过类名直接访问。