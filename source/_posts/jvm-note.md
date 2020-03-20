---
title: jvm 学习笔记
date: 2018-04-06 11:29:42
tags: jvm
categories: java
---
最近在看周志明先生的《深入理解JAVA虚拟机JVM高级特性与最佳实践》，作笔记如下，以便自已复习。文中代码，大部分摘自书中。

JVM的基本结构：类加载子系统、方法区、JAVA堆、JAVA栈、本地方法区、程序计数器、直接内存、垃圾回收系统和执行引擎。

JVM的运行时数据区如下所示：
![](/img/jvm-runtime-data-schema.png "图片来源于网络，仅用于学习使用")

JAVA的NIO库允许程序使用直接内存，从而提高性能，通常直接内存速度会优于堆。读写频繁的场合，可以优先考虑使用。

lambda函数式编程的一个重要优点就是这样的程序天然地适合并行运行，这对JAVA语言在多核时代继续保持主流语言的地位有很大的帮助。

### 垃圾回收算法
1. 引用计数法：是古老而经典的垃圾收集算法，其核心就是在对象被其他对象所引用的时候计数器加1，而当引用失效时则减1。但是这种方式有非常严重的问题：无法处理循环引用的情况、还有的就是每次进行加减操作比较浪费系统性能。
2. 标记清除法：分为标记和清除两个阶段，这种方式也有非常大的漏洞弊端，就是空间碎片问题，垃圾回收后，空间不是连续的。
3. 复制算法：其核心思想就是将内存分为两块，每次只使用其中一块。在回收时，将正在使用的内存中的存留对象复制到未被使用的内存中去，之后清除之前正在使用的内存中的所有对象，反复去交换两个内存的角式。
4. 标记压缩法：是在标记清除法的基础上做了优化，把存活的对象压缩到内存的一端，然后进行垃圾清理（老年代中使用的回收方法）
5. 分代算法：根据对象的特点把内存分成N块，然后根据每个对象的特点使用不同的算法。对于新生代，它的回收频率很高，但是每次回收耗时都很短；而老年代回收频率较低，但是耗时相对较长，所以应该尽量减少老年代的GC。

### 确定对象是否已死的方法
1. 引用计数法；
2. 可达性分析算法GC Root.

### 垃圾回收器
1. 串行回收器： 使用单线程进行垃圾回收的回收器。每次回收时，串行回收器只有一个工作线程，对于并行能力较弱的计算机来说，串行回收器的专注性和独占性往往有更好的性能表现。可以在新生代和老年代使用。`-XX:+UseSerialGC`
2. 并行回收器： 在串行回收器的基础上作了改进，他可以使用多个线程同时进行垃圾回收，对于计算能力强的计算机而言，可以有效的缩短回收所需的实际时间。他只是简单的将串行回收器多线程化，他的回收策略和算法和串行回收器一样。只使用在新生代。`--XX:+UseParNewGC`
3. ParallelGC： 新生代回收器，使用了复杂算法的收集器，也是多线程独占形式的收集器，它有个特点，就是它非常关注系统的吞吐量。
4. CMS：全称为`Concurrent Mark Sweep`，意为并发标记清除，他使用的是标记清除法，主要关注系统停顿时间。`-XX:+UseConcMarkSweepGC`进行设置，`-XX:ConcGCThreads`设置并发线程数量。CMS并不是独占的回收器，也就是说CMS回收的过程中，应用程序仍然可以在不停的工作，不过会有新的垃圾产生。CMS比较耗内存，CMS不会等到应用程序饱和的时候才去回收垃圾，而是在某一个阀值的时候开始回收，可以使用参数指定：`-XX:CMSInitiatingOccupancyFraction`来指定，默认为68,也就是说当老年代的空间使用率达68%的时候，会执行CMS回收。如果内存不足，收集可能会失败，如果失败了，会启动老年代串行回收器。
4. G1：`Garbage-First`是在JDK1.7中提出的垃圾回收器，是为了取代CMS的回收器。它属于分代回收器，并行性和并发性。并行性是G1回收期间可多线程同时工作，而并发性是G1拥有与应用程序交替执行能力，部分工作可与应用程序同时执行，在整个GC期间不会完全阻塞应用程序。它可以工作在新生代和老年代。之前的回收器，或者工作在新生代，或者工作在老年代。G1使用了有益智复制对象的方式，减少空间碎片。

将GC的日志输出到文件可以配置JVM启动参数：`-Xloggc:/home/hewentian/Document/gc.log`

### class文件格式
class文件是一组以8位字节码为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在class文件之中，中间没有添加任何分隔符，这使得整个class文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在。当遇到需要占用8位字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8位字节进行存储。

无符号数属于基本的数据类型，以u1, u2, u4, u8来分别代表1，2，4，8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。

如下图所示：
![](/img/class-file-format.png "图片来源：周志明先生的《深入理解JAVA虚拟机JVM高级特性与最佳实践》")

### 魔数与class文件的版本
**魔数：** 每个class文件的头4个字节称为魔数(Magic Number)，它的唯一作用是确定这个文件是否为一个能被虚拟机接受的class文件。
**版本号：** 紧接着魔数的4个字节存储的是class文件的版本号：第5和第6个字节是次版本号(Minor Version)，第7和第8个字节是主版本号(Major Version)。只有当前JVM的版本号大于等于这个文件的版本号，该文件才会被JVM加载。

### 类型转换
JVM直接支持（转换时无需显式的转换指令）：

1. int类型到long, float或者double类型；
2. long类型到float, double类型；
3. float类型到double类型。

### 类的加载过程
JVM中类的加载过程：加载、验证、准备、解析、初始化。如果算上：使用和缷载这两个过程，则一共有7个过程。

加载：加载要完成以下三件事：

1. 通过一个类的全限定名来获取定义此类的二进制字节流（不一定从文件中读取，可以从网络或数据库中）；
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构;
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

验证：验证是连接阶段的第一步，这一阶段的目的是为了确保class文件的字节流中包含的信息符合当前JVM的要求，并且不会危害JVM自身的安全。
准备：正式为类变量（static修饰）分配内存并设置类变量初始值的阶段，通常是该类型的零值；
``` java
public static int value = 123;
```
变量value在准备阶段过后的初始值为0，而不是123。赋值为123在初始化阶段才会执行。

解析：JVM将常量池内的符号引用替换为直接引用的过程；
初始化：执行类构造器的过程，设置实例变量的初始值。

其中，加载、验证、准备、初始化和缷载这5个阶段的顺序是确定的。如下图所示：
![](/img/class-loading.png "图片来源：周志明先生的《深入理解JAVA虚拟机JVM高级特性与最佳实践》")


### 类加载器
从JAVA开发人员的角度来看，绝大部分JAVA程序都会使用到以下3种系统提供的类加载器：

1. 启动类加载器(Bootstrap ClassLoader)：这个类加载器负责将存放在`<JAVA_HOME>/lib`目录中的，或者被`-Xbootclasspath`参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被JAVA程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给引导类加载器，那么使用null代替即可。
2. 扩展类加载器(Extension ClassLoader)：这个加载器由`sun.misc.Launcher$ExtClassLoader`实现，它负责加载`<JAVA_HOME>/lib/ext`目录中的，或者被`java.ext.dirs`系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。
3. 应用程序类加载器(Application ClassLoader)：这个类加载器由`sun.misc.Launcher$AppClassLoader`实现。由于这个类加载器是`ClassLoader中的getSystemClassLoader()`方法的返回值，所以一般也称它为系统类加载器。它负责加载用户类路径(ClassPath)上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自已的类加载器，一般情况下这个就是程序中默认的类加载器。

我们的应用程序都是由这3种类加载器互相配合进行加载的，如果有必要，还可以加入自已定义的类加载器。这些类加载器之间的关系一般如下图所示，这种层次关系，称为类加载器的双亲委派模型。
![](/img/class-loader-mode.png "图片来源：周志明先生的《深入理解JAVA虚拟机JVM高级特性与最佳实践》")

什么时候触发类加载：
1. `new`一个类实例的时候；
2. 调用一个类的`static`变量或方法的时候，例如：`System.out`；
3. 反射调用的时候，如果该类还没进行过初始化；
4. 初始化一个类，其父类还没初始化时，会先初始化其父类；
5. 启动时的类，即运行`main`方法类。

静态加载和动态加载：
静态加载：在代码中通过`new`创建实例，称为静态加载；
动态加载：在运行时，通过`Class.forName()`加载一个类，称为动态加载。

`loadClass()`和`Class.forName()`的区别：
`ClassLoader.loadClass()`仅会将类加载到内存中，但不会实例化对象。而`Class.forName()`在加载之后，会实例化对象，也就是说它会返回一个对象。

`new`和`newInstance()`创建类的区别：
1. `newInstance()`必须保证这个类被加载；
2. `new`关键字，如果类没有被加载，那么就会先加载；
3. `new`可以调用类的任何构造方法，而`newInstance()`只能调用默认的无参构造方法；
4. `new`出来的对象是强类型的，效率高；`newInstance()`创建的对象是弱类型的，效率相对较低。


### 当泛型遇上重载
下面的代码是不能通过编译的，因为参数`List<String>`和`List<Integer>`编译之后都被擦除了，变成了一样的`List<E>`，擦除动作导致这两种方法的特征签名变得一模一样。
``` java
public static void method(List<String> list) {
    System.out.println("invoke method(List<String> list)");
}

public static void method(List<Integer> list) {
    System.out.println("invoke method(List<Integer> list)");
}
```

如果真是要重载，可以适当修改上述代码，只要增加返回值即可，它也只能在JDK1.6上可以编译通过。在JDK1.8也是无法编译通过的。如下所示：
``` java
public static String method(List<String> list) {
    System.out.println("invoke method(List<String> list)");
    return "";
}

public static int method(List<Integer> list) {
    System.out.println("invoke method(List<Integer> list)");
    return 1;
}
```

### 自动装箱的陷阱
``` java
public static void main(String[] args) throws Exception {
    Integer a = 1;
    Integer b = 2;
    Integer c = 3;
    Integer d = 3;
    Integer e = 321;
    Integer f = 321;
    Long g = 3L;

    // 在 JDK1.8 中的结果如下
    System.out.println(c == d);             // true, [-127, 128]之间的Integer数字会被缓存
    System.out.println(e == f);             // false
    System.out.println(c == (a + b));       // true
    System.out.println(c.equals(a + b));    // true
    System.out.println(g == (a + b));       // true
    System.out.println(g.equals(a + b));    // false
}
```

### final语言校验
下面这两段代码编译出来的class文件是一样的，没有任何区别。只是在编写程序的时候会受到final的约束。
``` java
// 方法一：带有final修饰
public void foo(final int arg) {
    final int var = 0;
    // do something
}

// 方法二：没有final修饰
public void foo(int arg) {
    int var = 0;
    // do something
}
```

### 解释器与编译器
尽管不是所有的JVM都采用解释器与编译器并存的架构，但许多主流的商用JVM，如HotSpot、J9等，都同时包含解释器与编译器。**但是，三大商用JVM之一的JRockit是个例外，它内部没有解释器。**解释器与编译器两者各有优势：当程序需要迅速启动和执行的时候，解释器可以首先发挥作用，省去编译的时间，立即执行。在程序运行后，随着时间的推移，编译器逐渐发挥作用，把越来越多的代码编译成本地代码之后，可以获取更高的执行效率。当程序运行环境中内存资源限制较大（如部分嵌入式系统中），可以使用解释执行节约内存，反之可以使用编译执行来提升效率。在整个JVM执行架构中，解释器与编译器经常配合工作，如下图。
![](/img/compiler-and-interpreter.png "图片来源：周志明先生的《深入理解JAVA虚拟机JVM高级特性与最佳实践》")


### 程序编译与代码优化
从sun javac的代码来看，编译过程大致可以分成3个过程：

1. 解析与填充符号表的过程(Parse and Enter);
2. 插入式注解处理器的注解处理过程(Annotation Processing);
3. 分析与字节码生成的过程(Analyse and Generate)。

### 编译对象与触发条件
在程序运行的过程中会被即时编译器JIT编译的“热点代码”有两类：

1. 被多次调用的方法；
2. 被多次执行的循环体。

判断一段代码是不是热点代码，是不是需要触发即时编译，这样的行为称为热点探测(Hot Spot Detection)，其实进行热点探测并不一定要知道方法具体被调用了多少次，目前主要的热点探测判定方法有两种，分别如下：

* 基于采样的热点探测(Sample Based Hot Spot Detection)：采用这种方法的JVM会周期性地检查各个线程的栈顶，如果发现某个（或某些）方法经常出现在栈顶，那这个方法就是“热点方法”。基于采样的热点探测的好处是实现简单、高效，还可以很容易地获取方法调用关系（将调用堆栈展开即可），缺点是很难精确地确认一个方法的热度，容易因为受到线程阻塞或别的外界因素的影响而扰乱热点探测。
* 基于计数器的热点探测(Counter Based Hot Spot Detection)：采用这种方法的JVM会为每个方法（甚至是代码块）建立计数器，统计方法的执行次数，如果执行次数超过一定的阈值就认为它是“热点方法”。这种统计方法实现起来麻烦一些，需要为每个方法建立并维护计数器，而且不能直接获取到方法的调用关系，但是它的统计结果相对来说更加精确和严谨。


在HotSpot虚拟机中使用的是第二种--基于计数器的热点探测方法，因此它为每个方法准备了两类计数器：方法调用计数器(Invocation Counter)和回边计数器(Back Edge Counter)。

在确定虚拟机运行参数的前提下，这两个计数器都有一个确定的阈值，当计数器超过阈值溢出了，就会触发JIT编译。

我们首先来看看方法调用计数器，顾名思义，这个计数器就用于统计方法被调用的次数，它的默认阈值在Client模式下是1500次，在Server模式下是10000次，这个阈值可以通过JVM参数`-XX:CompileThreshold`来人为设定。

下面我们再来看看回边计数器。**什么是回边？ 在字节码遇到控制流向后跳转的指令称为回边（Back Edge）。**回边计数器是用来统计一个方法中循环体代码执行的次数，回边计数器的阈值可以通过参数`-XX：OnStackReplacePercentage`来调整。

虚拟机运行在Client模式下，回边计数器阈值计算公式为：
方法调用计数器阈值(CompileThreshold) x OSR比率(OnStackReplacePercentage) / 100
其中OnStackReplacePercentage默认值为933，如果都取默认值，那Client模式虚拟机的回边计数器的阈值为13995.

虚拟机运行在Server模式下，回边计数器阈值的计算公式为：
方法调用计数器阈值(CompileThreshold) x (OSR比率(OnStackReplacePercentage) - 解释器监控比率(InterpreterProfilePercentage) / 100
其中OnStackReplacePercentage默认值为140，InterpreterProfilePercentage默认值为33。
如果都取默认值，那Server模式虚拟机回边计数器的阑值为10700。

回边计数器与方法调用计数器不同的是，回边计数器没有热度衰减，因此这个计数器统计的就是循环执行的绝对次数。

参见下图：
![](/img/method-invoke-trigger-jit.png "图片来源：周志明先生的《深入理解JAVA虚拟机JVM高级特性与最佳实践》")


### 程序延时一定时间
像下面的空循环经JIT编译器优化后，会被消除掉，以前很多入门教程把空循环当做程序延时的手段来介绍，其实是错误的。
``` java
for(int i = 1; i <= 10000; i++)
    ;
```

要延时应该使用下面的方法
``` java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```


不使用的对象应手动设为null，不过，赋值为null的操作在经过JIT编译优化后就会被消除掉，这时候将变量设置为null就是没有意义的。


### 线程、主内存、工作内存、处理器的关系图
![](/img/cpu-thread-memory.png "图片来源：周志明先生的《深入理解JAVA虚拟机JVM高级特性与最佳实践》")


### 内存间交互操作
关于主内存与工作内存之间具体的交互协议，即一个变量如何从主内存复制到工作内存、如何从工作内存同步回主内存之类的实现细节，JAVA内存模型中定义了以下8种操作来完成，虚拟机实现时必须保证下面提及的每一种操作都是原子的、不可再分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许有例外）。

* lock（锁定）：作用于主内存的变量，它把一个变量标识为一条线程独占的状态；
* unlock（解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定；
* read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用；
* load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中；
* use（使用）：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作；
* assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作；
* store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用；
* write（写入）：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。


### volatile变量的使用场景
由于volatile变量只能保证可见性，在不符合以下两条规则的运算场景中，我们仍然要通过加锁（使用synchronized或java.util.concurrent中的原子类）来保证原子性。

1. 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值；
2. 变量不需要与其他的状态变量共同参与不变约束。

像下面的代码就很适合使用volatile变量来控制并发，当shutdown方法被调用时，能保证所有线程中执行的doWork()方法都立即停下来。
``` java
volatile boolean shutdownRequest;

public void shutdown() {
    shutdownRequest = true;
}

while(!shutdownRequest) {
    // do stuff
}
```


### 线程状态转换
Java语言定义了5种线程状态，在任意一个时间点，一个线程只能有且只有其中的一种状态，这5种状态分别如下：

* 新建（New）：创建后尚未启动的线程处于这种状态；
* 运行（Runnable）：Runnable包括了操作系统线程状态中的Running和Ready，也就是处于此状态的线程有可能正在执行，也有可能正在等待着CPU为它分配执行时间；
* 无限期等待（Waiting）：处于这种状态的线程不会被分配CPU执行时间，它们要等待被其他线程显式地唤醒。以下方法会让线程陷入无限期的等待状态：
    * 没有设置Timeout参数的Object.wait()方法；
    * 没有设置Timeout参数的Thread.join()方法；
    * LockSupport.park()方法；
* 限期等待（Timed Waiting）：处于这种状态的线程也不会被分配CPU执行时间，不过无须等待被其他线程显式地唤醒，在一定的时间之后它们会由系统自动唤醒。以下方法会让线程进入限期等待状态：
    * Thread.sleep()方法；
    * 设置了Timeout参数的Object.wait()方法；
    * 设置了Timeout参数的Thread.join()方法；
    * LockSupport.parkNanos()方法；
    * LockSupport.parkUntil()方法；
* 阻塞（Blocked）：线程被阻塞了，“阻塞状态”与“等待状态”的区别是：“阻塞状态”在等待着获取到一个排他锁，这个事件将在另外一个线程放弃这个锁的时候发生；而“等待状态”则是在等待一段时间，或者唤醒动作的发生。在程序等待进入同步区域的时候，线程将进入这种状态。
* 结束（Terminated）：已终止线程的线程状态，线程已经执行结束。

上述5种状态在遇到特定事件发生的时候会互相转换，它们的转换关系如下图所示：
![](/img/thread-state-transform.png "图片来源：周志明先生的《深入理解JAVA虚拟机JVM高级特性与最佳实践》")


### Java生成Heap Dump及OOM问题排查
引用自 https://www.baeldung.com/java-heap-dump-capture
A heap dump is a snapshot of all the objects that are in memory in the JVM at a certain moment. They are very useful to troubleshoot memory-leak problems and optimize memory usage in Java applications.

Heap dumps are usually stored in binary format hprof files. We can open and analyze these files using tools like jhat or JVisualVM. Also, for Eclipse users it's very common to use MAT.

**1. jmap**
jmap is a tool to print statistics about the memory in a running JVM. We can use it for local or remote processes.

To capture a heap dump using jmap we need to use the dump option:

        jmap -dump:[live],format=b,file=<file-path> <pid>

Along with that option, we should specify several parameters:

* live: if set it only prints objects which have active references and discards the ones that are ready to be garbage collected. This parameter is optional
* format=b: specifies that the dump file will be in binary format. If not set the result is the same
* file: the file where the dump will be written to
* pid: id of the Java process

An example would be like this:

        jmap -dump:live,format=b,file=/tmp/dump.hprof 12587

Remember that we can easily get the pid of a Java process by using the jps command.
Keep in mind that jmap was introduced in the JDK as an experimental tool and it's unsupported. Therefore, in some cases, it may be preferable to use other tools instead.

**2. jcmd**
jcmd is a very complete tool which works by sending command requests to the JVM. We have to use it in the same machine where the Java process is running.

One of its many commands is the GC.heap_dump. We can use it to get a heap dump just by specifying the pid of the process and the output file path:

        jcmd <pid> GC.heap_dump <file-path>

We can execute it with the same parameters that we used before:

        jcmd 12587 GC.heap_dump /tmp/dump.hprof

As with jmap, the dump generated is in binary format.

**3. JVisualVM**
JVisualVM is a tool with a graphical user interface that lets us monitor, troubleshoot and profile Java applications. The GUI is simple but very intuitive and easy to use.

One of its many options allows us to capture a heap dump. If we right-click on a Java process and select the “Heap Dump” option, the tool will create a heap dump and open it in a new tab:

        jvisualvm

Notice that we can find the path of the file created in the “Basic Info” section.

**Capture a Heap Dump Automatically**
All the tools that we've shown in the previous sections are intended to capture heap dumps manually at a specific time. In some cases, we want to get a heap dump when a java.lang.OutOfMemoryError occurs so it helps us investigate the error.

For these cases, Java provides the HeapDumpOnOutOfMemoryError command-line option that generates a heap dump when a java.lang.OutOfMemoryError is thrown:

        java -XX:+HeapDumpOnOutOfMemoryError

By default, it stores the dump in a `java_pid<pid>.hprof` file in the directory where we're running the application. If we want to specify another file or directory we can set it in the HeapDumpPath option:

        java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=<file-or-dir-path>

When our application runs out of memory using this option, we'll be able to see in the logs the created file that contains the heap dump:

        java.lang.OutOfMemoryError: Requested array size exceeds VM limit
        Dumping heap to java_pid12587.hprof ...
        Exception in thread "main" Heap dump file created [4744371 bytes in 0.029 secs]
        java.lang.OutOfMemoryError: Requested array size exceeds VM limit
            at com.baeldung.heapdump.App.main(App.java:7)

In the example above, it was written to the java_pid12587.hprof file.

As we can see, this option is very useful and there is no overhead when running an application with this option. Therefore, it's highly recommended to use this option always, especially in production.

Finally, this option can also be specified at runtime by using the HotSpotDiagnostic MBean. To do so, we can use JConsole and set the HeapDumpOnOutOfMemoryError VM option to true:

**4. JMX**
The last approach that we'll cover in this article is using JMX. We'll use the HotSpotDiagnostic MBean that we briefly introduced in the previous section. This MBean provides a dumpHeap method that accepts 2 parameters:

* outputFile: the path of the file for the dump. The file should have the hprof extension
* live: if set to true it dumps only the active objects in memory, as we've seen with jmap before

In the next sections, we'll show 2 different ways to invoke this method in order to capture a heap dump.

**4.1. JConsole**
The easiest way to use the HotSpotDiagnostic MBean is by using a JMX client such as JConsole.

If we open JConsole and connect to a running Java process, we can navigate to the MBeans tab and find the HotSpotDiagnostic under com.sun.management. In operations, we can find the dumpHeap method that we've described before:

As shown, we just need to introduce the parameters outputFile and live into the p0 and p1 text fields in order to perform the dumpHeap operation.

**4.2. Programmatic Way**
The other way to use the HotSpotDiagnostic MBean is by invoking it programmatically from Java code.

To do so, we first need to get an MBeanServer instance in order to get an MBean that is registered in the application. After that, we simply need to get an instance of a HotSpotDiagnosticMXBean and call its dumpHeap method.

Let's see it in code:

        public static void dumpHeap(String filePath, boolean live) throws IOException {
            MBeanServer server = ManagementFactory.getPlatformMBeanServer();
            HotSpotDiagnosticMXBean mxBean = ManagementFactory.newPlatformMXBeanProxy(
            server, "com.sun.management:type=HotSpotDiagnostic", HotSpotDiagnosticMXBean.class);
            mxBean.dumpHeap(filePath, live);
        }

Notice that an hprof file cannot be overwritten. Therefore, we should take this into account when creating an application that prints heap dumps. If we fail to do so we'll get an exception:

        Exception in thread "main" java.io.IOException: File exists
        at sun.management.HotSpotDiagnostic.dumpHeap0(Native Method)
        at sun.management.HotSpotDiagnostic.dumpHeap(HotSpotDiagnostic.java:60)

**5. Conclusion**
In this tutorial, we've shown multiple ways to capture a heap dump in Java.

As a rule of thumb, we should remember to use the HeapDumpOnOutOfMemoryError option always when running Java applications. For other purposes, any of the other tools can be perfectly used as long as we keep in mind the unsupported status of jmap.

we can open the file in Java VisualVM by choosing `File -> Load` from the main menu.

**声明：**图片来源于网络，仅用于学习使用。

