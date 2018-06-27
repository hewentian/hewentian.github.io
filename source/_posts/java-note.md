---
title: java 学习笔记
date: 2017-11-16 19:58:45
tags: java
categories: java
---
### 想在`eclipse`中新建类的时候，自动在类的头部插入作者、时间等信息
方法是`Window -> Preferences -> Java -> Code Style -> Code Templates`在右则民展开`Code -> New Java files`点`Edit...`并加入如下代码
``` java
${filecomment}
${package_declaration}
/**
 * 
 * <p>
 * <b>${file_name}</b> 是 
 * </p>
 *
 * @author <a href="mailto:wentian.he@qq.com">hewentian</a>
 * @date ${currentDate:date('yyyy-MM-dd')} ${time}
 * @since JDK 1.8
 *
 */
${typecomment}
${type_declaration}
```

### Remote System Explorer Operation总是运行后台服务，卡死eclipse解决办法
eclipse后台进程在远程操作，右下角显示的“Remote System Explorer Operation”。折腾了半天，在Stack Overflow找到答案 [源地址](https://stackoverflow.com/questions/1631817/remote-system-explorer-operation-causing-freeze-for-couple-of-seconds)。解决方案如下：

	step1: Eclipse -> Preferences -> General -> Startup and Shutdown.
		-Uncheck RSE UI.

	step2: Eclipse -> Preferences -> Remote Systems.
		-Uncheck Re-open Remote Systems view to previous state.

Update your Eclipse to 4.3.1 (at least) due to a bug on previous version.

Restart Eclipse and its done.


1. 线程安全概念：当多个线程访问某一个类（对象或方法）时，这个类始终都能表现出正确的行为，那么这个类（对象或方法）就是线程安全的。
2. synchronized：可以在何意对象及方法上加锁，而加锁的这段段码称为“互斥区”或“临界区”。

当多个线程访问某个synchronized修饰的方法时，以排队的方式进行处理（这里的排除是按照CPU分配时间片的先后顺序而定的），一个线程想要执行这个方法时，首先是尝试获得锁，如果拿到锁，就执行；否则，就会不断的尝试去获得这把锁，直到拿到为止，而且是多个线程同时去竞争这把锁。在static方法上加上synchronized关键字，表示锁定.class类，类一级别的锁（独占.class类)

一个类中有两个方法都有synchronized修饰的话，多线程分别访问这两个方法都会进行等待，因为它们都共用同一个锁。如果其中一个没有synchronized修饰的话，多线程就可以同时访问，而不需等待。


### Integer.valueOf产生的死锁
``` java
static class SyncAddRunnable implements Runnable {
	int a;
	int b;

	public SyncAddRunnable(int a, int b) {
		super();
		this.a = a;
		this.b = b;
	}

	public void run() {
		synchronized (Integer.valueOf(a)) {
			synchronized (Integer.valueOf(b)) {
				System.out.println(a + b);
			}
		}
	}
}

public static void main(String[] args) throws InterruptedException, IOException {
		for (int i = 0; i < 100; i++) {
			new Thread(new SyncAddRunnable(1, 2)).start();
			new Thread(new SyncAddRunnable(2, 1)).start();
		}
	}
```
上面的程序会造成死锁，其原因是Integer.valueOf方法基于减少对象创建次数和节省内存的考虑，[-128, 127]之间的数字会被缓存（默认值，实际值取决于`java.lang.Integer.IntegerCache.high`参数的设置），当valueOf()方法传入的参数在这个范围之内，将直接返回缓存中的对象，也就是说，代码中调用了200次Integer.valueOf方法一共就只返回了2个不同的对象。假如在某个线程的两个synchronized块之间发生了线程切换，那就会出现线程A等待被线程B持有的Integer.valueOf(1)，而线程B又等待着线程A持有的Integer.valueOf(2)，结果就出现了死锁了。

### 只保留3位小数
``` java
java.text.NumberFormat numberFormat = java.text.NumberFormat.getNumberInstance();
numberFormat.setMaximumFractionDigits(3);
System.out.println(numberFormat.format(3.1415927d)); // 3.142
```

### 数字转成百分比
``` java
java.text.NumberFormat percentFormat = java.text.NumberFormat.getPercentInstance();
percentFormat.setMaximumFractionDigits(2); // 最大小数位数
percentFormat.setMinimumFractionDigits(2); // 最小小数位数
percentFormat.setMaximumIntegerDigits(2); // 最大整数位数，多了会截掉最前面的
percentFormat.setMinimumIntegerDigits(2); // 最小整数位数，不够会在最前面补0
System.out.println(percentFormat.format(3.1415927d)); // 14.16%
System.out.println(percentFormat.format(0.0415927d)); // 04.16%
```

### btrace的使用
要使用btrace，必须先到这里下载：
https://github.com/btraceio/btrace
下载之后，解压到指定目录，还要配置系统的环境变量
``` bash
$ pwd
/home/hewentian/Downloads

$ tar xf btrace-bin-1.3.11.tar.gz 
$ mv btrace-bin-1.3.11 /usr/local/btrace-bin-1.3.11/

# 修改环境变量
$ vi /etc/profile

# 在/etc/profile中增加以下内容
# add btrace
export BTRACE_HOME=/usr/local/btrace-bin-1.3.11
export PATH=$BTRACE_HOME/bin:$PATH

$ source /etc/profile
```
下面用一个例子，演示如何使用：
示例1：获取方法参数以及返回值
将下面的代码保存为BTraceTest.java
``` java
package com.hewentian;

/**
 * 
 * <p>
 * <b>BTtraceTest.java</b> 是
 * </p>
 *
 * @author <a href="mailto:wentian.he@qq.com">hewentian</a>
 * @date 2018-04-19 3:57:22 PM
 * @since JDK 1.8
 *
 */
public class BTraceTest {
	public static String whoAreYou(String name, int age) {
		try {
			Thread.sleep(5000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}

		return "I'm " + name + ", " + age;
	}

	public static void main(String[] args) {
		for (int i = 0; i < 10; i++) {
			String sayYou = whoAreYou("Lily", 23);
			System.out.println(sayYou);
		}
	}
}
```

下面的是测试代码，同样是JAVA代码，将其保存为Btrace.java
``` java
/**
 * 
 * <p>
 * <b>Btrace.java</b> 是 
 * </p>
 *
 * @author <a href="mailto:wentian.he@qq.com">hewentian</a>
 * @date 2018-04-19 4:02:37 PM
 * @since JDK 1.8
 *
 */
import static com.sun.btrace.BTraceUtils.*;
import com.sun.btrace.annotations.*;

@BTrace
public class Btrace {
	@OnMethod(clazz = "com.hewentian.BTraceTest", method = "whoAreYou", location = @Location(Kind.RETURN))
	public static void whoAreYou(String name, int age, @Return String result) {
		println("name: " + name);
		println("age: " + age);
		println(result);
	}
}

```

下面开始测试：
在一个console中执行如下命令，将程序启动
``` bash
$ javac -d . BTraceTest.java	# 编译
$ java com.hewentian.BTraceTest # 运行

I'm Lily, 23
I'm Lily, 23
I'm Lily, 23
I'm Lily, 23
btrace WARNING: Invalid 'libs' configuration [null]. Path '/usr/local/btrace-bin-1.3.11/build/btrace-libs' does not exist.
I'm Lily, 23
I'm Lily, 23
I'm Lily, 23
I'm Lily, 23
I'm Lily, 23
I'm Lily, 23
```

在另一个console中执行如下命令，查询上面的程序的PID，以便用于监控
``` bash
$ jps -v
4498 BTraceTest
4516 Jps -Denv.class.path=.:/usr/local/java/jdk1.8.0_102/lib:/usr/local/java/jdk1.8.0_102/jre/lib: -Dapplication.home=/usr/local/java/jdk1.8.0_102 -Xms8m
2614 org.eclipse.equinox.launcher_1.4.0.v20161219-1356.jar -Dosgi.requiredJavaVersion=1.8 -Xms40m -Dosgi.module.lock.timeout=10 -Xverify:none -Xmx1200m
```
可以看到，我们的程序BTraceTest的PID为4498，执行下面的程序，开启btrace
``` bash
$ btrace 4498 Btrace.java

name: Lily
age: 23
I'm Lily, 23
name: Lily
age: 23
I'm Lily, 23
name: Lily
age: 23
I'm Lily, 23
name: Lily
age: 23
I'm Lily, 23
name: Lily
age: 23
I'm Lily, 23
```

示例2：计算方法运行消耗的时间，这在实际中非常有用
将下面的代码保存为BtraceDuration.java
``` java
/**
 * 
 * <p>
 * <b>BtraceDuration.java</b> 是 
 * </p>
 *
 * @author <a href="mailto:wentian.he@qq.com">hewentian</a>
 * @date 2018-04-19 4:46:50 PM
 * @since JDK 1.8
 *
 */
import static com.sun.btrace.BTraceUtils.*;
import com.sun.btrace.annotations.*;

@BTrace
public class BtraceDuration {
	@OnMethod(clazz = "com.hewentian.BTraceTest", method = "whoAreYou", location = @Location(Kind.RETURN))
	public static void whoAreYou(@Duration long duration) {
		println(strcat("duration(ms): ", str(duration / 1000000))); // duration的单位是纳秒，要除以 1,000,000 才是毫秒
	}
}
```
同样是按示例1的流程执行即可。
``` java
$ jps
2614 org.eclipse.equinox.launcher_1.4.0.v20161219-1356.jar
5258 Jps
5243 BTraceTest

$ btrace 5243 BtraceDuration.java 
duration(ms): 5000
duration(ms): 5000
duration(ms): 5000
duration(ms): 5000
duration(ms): 5000
duration(ms): 5000
```
btrace的介绍就到这里，这只是入门。更多内容，参考了：http://calvin1978.blogcn.com/articles/btrace1.html。这篇文章写得非常好。

在我们对一个对象的方法加锁的时候，需要考虑业务的整体性，即为getter/setter方法同时加锁synchronized同步关键字，保证业务的原子性。以免出现脏读。

关键字synchronized拥有重入锁的功能，也就是说在使用synchronized时，当一个线程得到了一个对象的锁后，再次请求此对象时是可以再次得到该对象的锁。


同步方法直接在方法上加synchronized实现加锁，同步代码块则在方法内部加锁，很明显，同步方法锁的范围比较大，而同步代码块范围要小点，一般同步的范围越大，性能就越差，一般需要加锁进行同步的时候，肯定是范围越小越好，这样性能更好

ReentrantLock可重入锁的意思是，一个线程可以对已被加锁的ReentrantLock锁再次加锁，ReentrantLock对象会维持一个计数器来追踪lock()方法的嵌套调用，线程在每次调用lock()加锁后，必须显式调用unlock()来释放锁，所以一段被锁保护的代码可以调用另一个被相同锁保护的方法。

所以我们在用synchronized关键字的时候，能缩小代码段的范围就尽量缩小，能在代码段上加同步就不要再整个方法上加同步。这叫减小锁的粒度，使代码更大程度的并发。

wait notfiy 方法，wait释放锁，notfiy不释放锁。用这种方式会有不实时的坏处。因为虽然A线程发出了notify，但是它不释放锁，这样B线程还需要wait直到A执行完它的时间片。可以使用CountDownLatch来实现实时。

Object类的方法有：

1. public: notify(), notifyAll();
2. public: wait(), wait(long),wait(long, int);
3. public: toString(), equals(Object), hashCode(), getClass();
4. protected: clone(), finalize();
5. private: registerNatives().

动态类型语言的关键特征是它的类型检查的主体过程是在运行期而不是在编译期。

java性能测试工具jmeter：http://jmeter.apache.org/
和它的好搭档badboy：http://www.badboy.com.au/
jvisualvm




