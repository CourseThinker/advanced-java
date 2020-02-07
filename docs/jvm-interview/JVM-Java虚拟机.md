# JVM-Java虚拟机

## 1.内存结构

### 1.1.程序计数器

#### 1.1.1.定义

Program Counter Register**程序计数器(寄存器)**

#### 1.1.2.作用

```.java
二进制指令      JVM指令	源代码
0:getstatic    #20      // PrintStream out = System.out;
3:astore_1           	// --
4:aload_1				// out.println(1);
5:iconst_1     		    // --
6:invokevirtual#26  	// --
9:aload_1				// out.println(2);
10:iconst_2				// --
11:invokevirtual #26	// --
```

二进制指令--》解释器--》机器码--》CPU

作用：保存线程执行的Jvm指令地址，让线程下一次执行的时候知道执行的位置；

#### 1.1.3.特点

1.线程私有

2.不存在内存溢出

### 1.2.虚拟机栈

![](/images/Jvm-虚拟机栈.png)

#### 1.2.1.定义

**Java Virtual Machine Stacks (Java虚拟机栈)**

存储信息：参数、存储局部变量表、操作数栈、动态连接、方法返回地址；

1.每个线程运行时所需要的内存，称为虚拟机栈；

2.每个栈由多个栈帧(Frame)组成，对应着每次方法调用时所需要的内存；

3.每个线程只能有一个活动的栈帧，对应当前的执行方法；

#### 1.2.2.问题辨析

1.垃圾回收是否涉及栈内存？

**垃圾回收不会涉及栈内存**

2.栈内存分配越大越好吗？

栈默认的大小为1024KB--Linux，macOs，windows依赖虚拟内存；

并不是越大越好，越大只能说明能够更多的递归方法调用，但是会减少执行的线程数；比如：100M的内存，如果栈大小为1M则可以创建100条线程，如果栈大小为2M，则只能创建50条线程；

3.方法内的局部变量是否线程安全？

	* 如果方法内局部变量没有逃离方法的作用访问，则是线程安全的；
	* 如果局部变量引用了对象，并逃离了方法的作用范围，则不是线程安全的；

测试代码：

```.java
/**
 * 局部变量线程安全问题
 */
public class Stack_01 {
    public static void main(String[] args) {
        StringBuilder sb = new StringBuilder();
        sb.append(1);
        sb.append(2);
        // 方法m2和m3存在线程安全问题
        new Thread(() -> m2(sb)).start();
        new Thread(() -> m3(sb)).start();
    }
    // 不会出现线程安全的问题
    public static void m1() {
        StringBuilder sb = new StringBuilder();
        sb.append(1);
        sb.append(2);
        sb.append(3);
        System.out.println(sb.toString());
    }
    // 存在线程安全问题
    public static void m2(StringBuilder sb) {
        sb.append(1);
        sb.append(2);
        sb.append(3);
        System.out.println(sb.toString());
    }
    // 内存线程安全问题
    public static StringBuilder m3(StringBuilder sb) {
        sb.append(1);
        sb.append(2);
        sb.append(3);
        return sb;
    }
}
```



#### 1.2.3.栈内存溢出

1.方法递归调用，超过了内存的大小，例如：

```.java
/**
 * 栈内存溢出 Xss256k
 * java.lang.StackOverflowError
 */
public class Stack_02 {
    private static int i;

    public static void main(String[] args) {
        try {
            method();
        } catch (Throwable e) {
            e.printStackTrace();
            System.out.println("递归次数：" + i);
        }
    }

    public static void method() {
        i++;
        method();
    }
}
```

#### 1.2.4.线程运行诊断

案例一：cpu占用过多  --  死循环

Java代码：

```.java
package com.itcm.jvm;

/**
 * cpu占用很高 死循环
 *
 */
public class Stack_03 {
    private static int i;

    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            while (true) {
                System.out.println("死循环中.......");
            }
        });
        thread.setName("t_stack");
        thread.start();
    }
}
```



* 定位：

  1.用top定位哪个进程对cpu占用很高：top

  ![image-20200203182303138](/images/image-20200203182303138.png)

  2.用ps命令查看哪个线程占用cpu很高：

  ps H -eo pid,tid,%cpu | grep 进程ID   --  查看进程ID,线程ID,CPU占用情况  --  将十进制的线程ID转换为16进制的值，与jstack pid命令的结果对比，找到对应出现问题的代码；

  ![image-20200203182541149](/images/image-20200203182541149.png)

  3.jstack  进程ID   --  查看进程中的线程信息  可以根据线程ID找到有问题的代码信息；

  ![image-20200204132755068](/images/image-20200204132755068.png)

案例二：程序运行很长时间  -- 死锁(deadlock)

代码：

```.java
/**
 * cpu占用很高 死锁 -- deadlock
 *
 */
public class Stack_04 {

    static Object a = new Object();
    static Object b = new Object();

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (a) {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (b) {
                   System.out.println("线程1:" + Thread.currentThread().getName());
               }
            }
        }).start();
        TimeUnit.SECONDS.sleep(1);
        new Thread(() -> {
            synchronized (b) {
                synchronized (a) {
                    System.out.println("线程2:" + Thread.currentThread().getName());
                }
            }
        }).start();
    }
}
```

![image-20200204141234652](/images/image-20200204141234652.png)

### 1.3.本地方法栈 -- Native Methed Stacks

### 1.4.堆 -- Heap

#### 1.4.1.定义

* 通过new关键字，创建对象都会使用堆内存

特点

* 线程共享的，堆中对象都需要考虑线程安全问题
* 有垃圾回收机制

#### 1.4.2.堆内存溢出

测试代码：

```.java
/**
 * 堆内存溢出  --- java.lang.OutOfMemoryError: Java heap space
 * -Xmx10m
 */
public class Heap_01 {
    public static void main(String[] args) {
        int i = 0;
        try {
            ArrayList<Object> list = Lists.newArrayList();
            String str = "hello heap";
            while (true) {
                str = str + str;
                i++;
                list.add(str);
            }
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            System.out.println(i);
        }
    }
}
```

异常信息：

```.java
java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:448)
	at java.lang.StringBuilder.append(StringBuilder.java:136)
	at com.itcm.jvm.Heap_01.main(Heap_01.java:18)
16
```



#### 1.4.2.堆内存诊断

1.jps工具

* 查看当前系统中有哪些java进程

2.jmap工具

* 查看堆内存使用情况 jmap -heap 进程ID

* C:\Program Files\Java\jdk1.8.0_131\bin>jmap -heap 5784
  Attaching to process ID 5784, please wait...
  Debugger attached successfully.
  Server compiler detected.
  JVM version is 25.131-b11

  using thread-local object allocation.
  Parallel GC with 4 thread(s)

  Heap Configuration:
     MinHeapFreeRatio         = 0
     MaxHeapFreeRatio         = 100
     MaxHeapSize              = 1038090240 (990.0MB)
     NewSize                  = 21495808 (20.5MB)
     MaxNewSize               = 346030080 (330.0MB)
     OldSize                  = 43515904 (41.5MB)
     NewRatio                 = 2
     SurvivorRatio            = 8
     MetaspaceSize            = 21807104 (20.796875MB)
     CompressedClassSpaceSize = 1073741824 (1024.0MB)
     MaxMetaspaceSize         = 17592186044415 MB
     G1HeapRegionSize         = 0 (0.0MB)

  Heap Usage:
  PS Young Generation
  Eden Space:
     capacity = 16252928 (15.5MB)
     used     = 525008 (0.5006866455078125MB)
     free     = 15727920 (14.999313354492188MB)
     3.2302364226310485% used
  From Space:
     capacity = 2621440 (2.5MB)
     used     = 0 (0.0MB)
     free     = 2621440 (2.5MB)
     0.0% used
  To Space:
     capacity = 2621440 (2.5MB)
     used     = 0 (0.0MB)
     free     = 2621440 (2.5MB)
     0.0% used
  PS Old Generation
     capacity = 43515904 (41.5MB)
     used     = 1024416 (0.976959228515625MB)
     free     = 42491488 (40.523040771484375MB)
     2.354118622929217% used

  1740 interned Strings occupying 157152 bytes.

\#dump jvm二进制的内存详细使用情况 （效果同在Tomcat的catalina.sh中添加 set JAVA_OPTS=%JAVA_OPTS% -server -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/heapdump  此文件需要借用内存分析工具如：Memory Analyzer (MAT)来分析）

 **jmap -dump:format=b,file=/home/$pid/jmapdump.txt $pid**

3.jconsole工具

* 图形化界面，连续监控

4.jvisualvm工具

### 1.5.方法区

#### 1.5.1.定义

#### 1.5.2.组成

![](/images/jvm-方法区.png)

#### 1.5.3.方法区内存溢出

* 1.8以前会导致永久代内存溢出

  ```.xml
  * 演示永久代内存溢出  java.lang.OutOfmemoryError：PerGen space
  * -XX:MaxPermSize=8m
  ```

* 1.8之后会导致元空间内存溢出

  ```.xml
  * 演示元空间内存溢出  java.lang.OutOfmemoryError：Metaspace
  * -XX:MaxMetaspaceSize=8m
  ```

  

代码示例：

```.java
/**
 * 方法区 内存溢出  Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
 * -XX:MaxMetaspaceSize=10m 设置元空间参数
 * 加载类的二进制字节码
 */

public class MethodArea_01 extends ClassLoader {

    public static void main(String[] args) {
        int j = 0;
        try {
            MethodArea_01 test = new MethodArea_01();
            for (int i = 0; i< 10000; i++, j++) {
                // 生成类的二进制字节码
                ClassWriter cw = new ClassWriter(0);
                // 版本号，public，类名，包名，父类，接口
                cw.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "Class_" + i, null,"java/lang/Object", null);
                // 返回byte[]
                byte[] bytes = cw.toByteArray();
                // 执行类的加载
                test.defineClass("Class_" + i, bytes, 0, bytes.length);
            }
        } catch (Exception e) {

        } finally {
            System.out.println(j);
        }
    }
}
```

异常信息：

```.java
8011
Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
	at com.itcm.jvm.MethodArea_01.main(MethodArea_01.java:26)
```

#### 1.5.4.运行时常量池

* 常量池，就是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等信息；
* 运行时常量池，常量池是*.class文件中的，当该类被加载，它的常量池信息就会放入运行时常量池，并把里面的符号地址变为真实地址；

#### 1.5.5.运行时常量池面试题 -- StringTable

```java
public class StringTable_01 {
    /**
     * StringTable ["a", "b", "ab"]  -- HashTable结构，不能扩容
     * 常量池中的信息，都会被加载到运行时常量池中，这时a，b，ab都是常量池中的符号，还没有变为java字符串对象
     * @param args
     */
    public static void main(String[] args) {
        String s1 = "a"; // 惰性的
        String s2 = "b";
        String s3 = "a" + "b"; // 在编译期间被优化，结果已经为ab
        // new StringBuilder.append("a").append("b").toString(); -->new String(value, 0, count); 存在于堆中
        String s4 = s1 + s2;
        String s5 = "ab";
        String s6 = s4.intern();

        // 面试题 问？
        System.out.println("s3==s4:" + (s3 == s4)); // false
        System.out.println("s3==s5:" + (s3 == s5)); // true
        System.out.println("s3==s6:" + (s3 == s6)); // true
        System.out.println("s5==s6:" + (s5 == s6)); // true
 
        String x1 = new String("c") + new String("d");
        String x2 = "cd";
        //x2.intern();

        System.out.println("x1==x2:" + (x1 == x2)); // false
    }
}
```

#### 1.5.6.字符串常量池 -- StringTable

* 常量池中的字符串仅仅是符号，第一次用到时才变为对象
* 利用串池的机制，来避免重复创建字符串对象
* 字符串变量拼接的原理是StringBuilder  （JDK 1.8）
* 字符串常量拼接的原理是编译器优化
* 可以使用intern方法，主动将串池中还没有的字符串放入字符串常量池中
  * JDK1.8 将字符串对象尝试放入字符串常量池中，如果存在不会放入，不存在则放入,会把常量池中的对象返回
  * JDK1.6 将字符串对象尝试放入字符串常量池中，如果存在不会放入，不存在则把此对象复制一份放入字符串常量池中，本身不变,会把常量池中的对象返回

```java
public class StringTable_02 {
    public static void main(String[] args) {
        // StringTable ["a", "b"]  -- HashTable结构，不能扩容
        // 创建的a、b在字符串常量池中,new String("ab") -- 堆中
        String s = new String("a") + new String("b");
        String intern = s.intern();// 将字符串对象尝试放入字符串常量池中，如果存在不会放入，不存在则放入,会吧常量池中的对象返回
        System.out.println(intern == "ab"); // true
    }
}
```

#### 1.5.7.字符串常量池 -- StringTable位置

* jdk1.8 存在于堆中
* jdk1.6 存在于永久代中

```java
/**
 * 演示StringTable位置
 * jdk8 -Xmx10m -XX:-UseGCOverheadLimit
 * 不设置 第二个参数，异常信息为：
 * Connected to the target VM, address: '127.0.0.1:52540', transport: 'socket'
 * 142095
 * Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
 * 	at java.lang.Integer.toString(Integer.java:403)
 * 	at java.lang.String.valueOf(String.java:3099)
 * 	at com.itcm.jvm.StringTable_03.main(StringTable_03.java:18)
 * 	超过98%的时间用来做GC并且回收了不到2%的堆内存时会抛出此异常
 * jdk6 -XX:MaxPermSize=10m  设置永久代大小
 */
public class StringTable_03 {
    public static void main(String[] args) {
        List<String> list = Lists.newArrayList();
        int i = 0;
        try {
            for (int j = 0; j < 260000; j++) {
                list.add(String.valueOf(j).intern());
                i++;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }
    }
}

```

#### 1.5.8.StringTable--垃圾回收

```java
/**
 * 演示StringTable 垃圾回收
 * jdk8 -Xmx10m
 * -XX:+PrintStringTableStatistics 在JVM进程退出时，打印出StringTable的统计信息输出到gclog中
 * -XX:+PrintGCDetails  打印gc信息
 * -verbose:gc  表示输出虚拟机中GC的详细情况
 * -XX:StringTableSize=200000  设置字符串常量池的大小
 * StringTable statistics:
 * Number of buckets       :     60013 =    480104 bytes, avg   8.000  // 容量
 * Number of entries       :      1839 =     44136 bytes, avg  24.000  // 存储的数据
 * Number of literals      :      1839 =    161904 bytes, avg  88.039
 * Total footprint         :           =    686144 bytes
 * Average bucket size     :     0.031
 * Variance of bucket size :     0.031
 * Std. dev. of bucket size:     0.175
 * Maximum bucket size     :         2
 */
public class StringTable_04 {
    public static void main(String[] args) {
        String s = "a";
        try {
            for (int i = 0; i < 100000; i++) {
                System.out.println(String.valueOf(i).intern());
            }
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
        }
    }
}
```

#### 1.5.9.StringTable--性能调优

* 调整 -XX:StringTabelSize=桶个数
* 考虑将字符串对象是否入池

### 1.6.直接内存

#### 1.6.1.定义

Direct Memory

* 常见于NIO操作时，用于数据缓冲区
* 分配回收成本较高，但读写性能高
* 不受JVM内存回收管理

传统IO：

![image-20200205154959550](/images/image-20200205154959550.png)

NIO模型：

![image-20200205155109656](/images/image-20200205155109656.png)

#### 1.6.2.内存溢出

```java
/**
 * 直接内存溢出
 * Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
 *     at java.nio.HeapByteBuffer.<init>(HeapByteBuffer.java:57)
 *     at java.nio.ByteBuffer.allocate(ByteBuffer.java:335)
 *     at com.itcm.jvm.DirectMemory_01.main(DirectMemory_01.java:18)
 */
public class DirectMemory_01 {

    private static final int _100M = 1024 * 1024 * 100;

    public static void main(String[] args) {
        ArrayList<ByteBuffer> list = Lists.newArrayList();
        int j = 0;
        try {
            for (int i = 0; i < 100; i++, j++) {
                ByteBuffer buffer = ByteBuffer.allocate(_100M);
                list.add(buffer);
            }
        } catch (Throwable e) {
            e.printStackTrace();
            System.out.println(j);
        }
    }
}
```

#### 1.6.3.分配和回收原理

* 使用了Unsafe对象完成直接内存的分配回收，并且回收需要主动调用freeMemory
* ByteBuffer的实现类内部，使用了Cleaner(虚引用)来监测ByteBuffer对象，一旦ByteBuffer对象被垃圾回收，那么就会由ReferenceHanlder线程通过Cleaner的clean方法调用freeMemory来释放直接内存

## 2.垃圾回收

### 2.1.如何判断对象可以回收

#### 2.1.1.引用计数法

问题：循环引用，不能被回收

#### 2.1.2.可达性分析算法

* Java虚拟机中的垃圾收集器采用可达性分析来探索所有存活的对象
* 扫描堆中的对象，看是否能够沿着Gc Root对象为起点的引用链找到该对象，找不到，表示可以回收
* 哪些对象可以作为GC Root？
  * 虚拟机栈中，局部变量引用的对象
  * 方法区，类静态属性的引用
  * 方法区，常量对象的引入
  * 本地方法栈，Native方法对象的引入
  * 加锁对象的引入

#### 2.1.3.四种引用

1.强引用

* 只有所有GC Roots对象都不通过【强引用】引用该对象，该对象才能被垃圾回收

2.软引用(SoftReference)

* 仅有软引用引用该对象时，在垃圾回收后，内存仍不足时会再次触发垃圾回收，回收软引用对象
* 可以配合引用队列释放软引用自身

3.弱引用(WeakReference)

* 仅有弱引用引用该对象时，在垃圾回收时，无论内存是否充足，都会回收弱引用对象
* 可以配合引用队列来释放弱引用自身

4.虚引用(PhantomReference)

* 必须配合引用队列使用，主要配合ByteBuffer引用，被引用对象回收时，会将虚引用入队，由Reference Handler线程调用虚引用相关方法释放直接内存

5.终结器引用(FinalReference)

* 无需手动编码，但其内部配合引用队列使用，在垃圾回收时，终结器引用入队(被引用对象暂时没有被回收)，

  在由Finalizer线程通过终结器引用找到被引用对象并调用它的finalize方法，第二次gc时才能被回收

```java
/**
 * 软引用测试 -Xmx20m
 */
public class SoftReference_01 {

    private static final int _4M = 1024 * 1024 * 4;

    public static void main(String[] args) {
//        strong();
        soft();
    }

    // 强引用 -- 抛出异常java.lang.OutOfMemoryError: Java heap space
    public static void strong() {
        List<byte[]> list = Lists.newArrayList();
        for (int i = 0; i < 5; i++) {
            list.add(new byte[_4M]);
        }
    }

    // 弱引用 -- 不会抛出异常
    public static void soft() {
        List<SoftReference<byte[]>> list = Lists.newArrayList();
        for (int i = 0; i < 5; i++) {
            SoftReference<byte[]> softReference = new SoftReference<>(new byte[_4M]);
            System.out.println(softReference.get());
            list.add(softReference);
            System.out.println(list.size());
        }
        System.out.println("循环结束:" + list.size());
        for (SoftReference<byte[]> reference : list) {
            System.out.println(reference.get());
        }
    }
}
```

### 2.2.垃圾回收算法

#### 2.2.1.标记清除

![image-20200205180319941](/images/image-20200205180319941.png)

过程：1.标记没有Gc Roots引用的对象 2.清除没有使用的内存空间

缺点：**产生内存碎片，如果下次要分配较大的内存，则不能分配**

#### 2.2.2.标记整理

![image-20200205180702167](/images/image-20200205180702167.png)

过程：1.标记没有被Gc Roots引用的对象 2.整理：将没有被回收的对象移动到一段，减少内存碎片

优缺点：**不会产生内存碎片，过程复制，需要更多的时间，比如修改引用对象的内存地址等**

#### 2.2.3.复制算法

![](/images/image-20200205181314101.png)

过程：1.标记没有被Gc Roots引用的对象 2.复制：将没有被回收的对象移动到另一块内存空间，清除之前的内存空间

优缺点：占用双倍内存空间

### 2.3.分代垃圾回收

#### 2.3.1.分代GC

![image-20200205193141048](/images/image-20200205193141048.png)



* 对象首先分布在iden(伊甸园)区域
* 新生代空间不足时，触发minor GC，伊甸园和from存活的对象使用copy算法复制到to survivor区中，存活的对象年龄+1，并且交换From Survivor和To Survivor
* minor gc会引发stop the world，暂停其他用户线程，等垃圾回收结束后，其他线程恢复运行
* 当对象寿命超过阈值时，会晋升到老年代，最大寿命为15(4bit)
* 当老年代空间不足时，会先触发minor gc，如果之后空间任然不足，则会触发full gc，暂停其他用户线程时间更长，full gc之后空间仍然不足，则抛出OutOfMemory异常

#### 2.3.2.JVM相关参数

| 含义             | 参数                                                       |
| ---------------- | ---------------------------------------------------------- |
| 堆初始化大小     | -Xms10m                                                    |
| 堆最大大小       | -Xmx512m或-XX:MaxHeapSize=512m                             |
| 新生代大小       | -Xmn10m或(-XX:NewSize=10m  \|  -XX:MaxNewSize=10m)         |
| 幸存区比例(动态) | -XX:InitialSurvivorRatio=8:1:1和-XX:+UseAdaptiveSizePolicy |
| 幸存区比例       | -XX:SurvivorRatio=8:1:1                                    |
| 晋升阈值         | -XX:MaxTenuringThreshold=15                                |
| 晋升详情         | -XX:+PrintTenuringDistribution                             |
| GC详情           | -XX:+PringGCDetails  -verbose:gc                           |
| FullGC前MinorGC  | -XX:+ScavengeBeforeFullGC                                  |
| 设置垃圾回收器   | -XX:+UseSerialGC                                           |

### 2.4.垃圾回收器

#### 2.4.1.串行

* 单线程
* 堆内存较小，适合个人电脑  -- cpu个数少
* -XX:+UseSerialGC = Serial(新生代)  + SerialOld(老年代)

![image-20200206012421714](/images/image-20200206012421714.png)

串行收集器组合Serial + Serial Old

```.xml
开启选项：-XX:+SerialGC
```

串行收集器是最基本、发展时间最长、久经考验的垃圾收集器，也是client模式下的默认收集器配置。

串行收集器采用单线程stop-the-world的方式进行收集。当内存不足时，串行GC设置停顿标识，待所有线程都进入安全点(Safepoint)时，应用线程暂停，串行GC开始工作，采用单线程方式回收空间并整理内存。单线程也意味着复杂度更低、占用内存更少，但同时也意味着不能有效利用多核优势。事实上，串行收集器特别适合堆内存不高、单核甚至双核CPU的场合。

#### 2.4.2.吞吐量优先 -- 并行

* 多线程
* 堆内存较大，多核cpu
* 让单位时间内，其他线程暂停时间最短

![image-20200206013329623](/images/image-20200206013329623.png)

并行收集器组合 Parallel Scavenge + Parallel Old  (JDK 7 8 默认垃圾收集器)

```.xml
开启选项：-XX:+UseParallelGC或-XX:+UseParallelOldGC(可互相激活)
```

并行收集器是以关注吞吐量为目标的垃圾收集器，也是server模式下的默认收集器配置，对吞吐量的关注主要体现在年轻代Parallel Scavenge收集器上。

并行收集器与串行收集器工作模式相似，都是stop-the-world方式，只是暂停时并行地进行垃圾收集。年轻代采用复制算法，老年代采用标记-整理，在回收的同时还会对内存进行压缩。关注吞吐量主要指年轻代的Parallel Scavenge收集器，通过两个目标参数`-XX:MaxGCPauseMills`和`-XX:GCTimeRatio`，调整新生代空间大小，来降低GC触发的频率。并行收集器适合对吞吐量要求远远高于延迟要求的场景，并且在满足最差延时的情况下，并行收集器将提供最佳的吞吐量。

#### 2.4.3.响应时间优先 -- 并发标记清除收集器

* 多线程
* 堆内存较大，多核cpu
* 尽可能单次内让其他线程暂停时间最短

![image-20200206014308925](/images/image-20200206014308925.png)

并发标记清除收集器组合 ParNew + CMS + Serial Old

```.xml
开启选项：-XX:+UseConcMarkSweepGC
```

并发标记清除(CMS)是以关注延迟为目标、十分优秀的垃圾回收算法，开启后，年轻代使用STW式的并行收集，老年代回收采用CMS进行垃圾回收，对延迟的关注也主要体现在老年代CMS上。

年轻代ParNew与并行收集器类似，而老年代CMS每个收集周期都要经历：**初始标记、并发标记、重新标记、并发清除**。其中，初始标记以STW的方式标记所有的根对象；并发标记则同应用线程一起并行，标记出根对象的可达路径；在进行垃圾回收前，CMS再以一个STW进行重新标记，标记那些由mutator线程(指引起数据变化的线程，即应用线程)修改而可能错过的可达对象；最后得到的不可达对象将在并发清除阶段进行回收。**值得注意的是，初始标记和重新标记都已优化为多线程执行**。CMS非常适合堆内存大、CPU核数多的服务器端应用，也是G1出现之前大型应用的首选收集器。

但是CMS并不完美，它有以下缺点：

1. 由于并发进行，CMS在收集与应用线程会同时会增加对堆内存的占用，也就是说，CMS必须要在老年代堆内存用尽之前完成垃圾回收，否则CMS回收失败时，将触发担保机制，串行老年代收集器将会以STW的方式进行一次GC，从而造成较大停顿时间；
2. 标记清除算法无法整理空间碎片，老年代空间会随着应用时长被逐步耗尽，最后将不得不通过担保机制对堆内存进行压缩。CMS也提供了参数`-XX:CMSFullGCsBeForeCompaction`(默认0，即每次都进行内存整理)来指定多少次CMS收集之后，进行一次压缩的Full GC。

### 2.5.垃圾收集器优化

预备知识

* 掌握GC相关的VM参数，会基本的空间调整
* 掌握相关工具
* 明白一点：调优跟应用、环境有关，没有放之四海而皆准的法则
* "C:\Program Files\Java\jdk1.8.0_131\bin\java" -XX:+PrintFlagsFinal -version | findstr "GC"  查看虚拟机运行参数

#### 2.5.1.调优领域

* 内存
* 锁竞争
* cpu占用
* io

#### 2.5.2.确定目标

* 【低延迟】还是【高吞吐量】，选择合适的回收器
* CMS，G1，ZGC
* ParallelGC
* Zing

#### 2.5.3.最快的GC是不发生GC

* 查看FullGC前后的内存占用，考虑下面几个问题
  * 数据是否太多？
    * resultSet = statement.executeQuery(select * from 大表 limit n)
  * 数据表示是否太臃肿
    * 对象图
    * 对象大小16 Integer 24 int 4
  * 是否存在内存泄漏
    * static Map map
    * 软引用
    * 弱引用
    * 分布式缓存 -- redis

#### 2.5.4.新生代调优

* 新生代特点
  * 所有的new操作的内存分配非常廉价
  * TLAB thread-local allocation buffer  -- 线程局部缓冲区
* 死亡对象的回收代价是零
* 大部分对象用过即死
* Minor GC的时间远远低于Full GC
* 新生代内存越大越好吗？

![image-20200206120739509](/images/image-20200206120739509.png)

* 新生代能够容纳【并发数*（请求-响应）】的内存容量

* 幸存区大到能保留【当前活跃对象+需要晋升的对象】

* 晋升阈值配置得当，让长时间存活对象尽快晋升

  * -XX:MaxTenuringThreshold=threshold   调整新生代到老年代的晋升阈值

  * -XX:+PrintTenuringDistribution  打印对象年代信息

    ![image-20200206122500423](/images/image-20200206122500423.png)

#### 2.5.5.老年代优化

以CMS为例

* CMS的老年代内存越大越好
* 先尝试不做调优，如果没有FullGC那么已经，否则先尝试调优新生代
* 观察发生Full GC时老年代内存占用，将老年代内存预设调大1/4~1/3
  * -XX:CMSInitiatingOccupancyFraction=percent

#### 2.5.6.案例分析

* 案例1 Full GC和Minor GC频繁
* 案例2 请求高峰期发生Full Gc，单次暂停时间特别长 CMS
* 案例3 老年代充裕情况下，发生Full GC JDK1.7

## 3.类加载机制

### 3.1.加载

* 将类的字节码载入方法区中，内部采用C++的instanceKlass描述java类，它的重要field有：

  * _java_mirror即java的类镜像，例如对String来说，就是String.class，作用是吧class暴露给java是使用
  * _super即父类
  * field即成员变量
  * _methods即方法
  * _constants即常量池
  * _class_loader即类加载器
  * _vtable虚方法表
  * _itable接口方法表

* 如果这个类还有父类没有加载，先加载父类

* 加载和链接可能是交替运动的

  ```xml
  **注意**
  *instanceClass这样的【元数据】是存储在方法区中的(jdk1.8在元空间)，但_java_mirror存在在堆中
  *可以通过前面介绍的HSDB工具查看
  ```

  ![image-20200206160340073](/images/image-20200206160340073.png)

### 3.2.链接

* 验证：验证类是否符合JVM规范，安全性检查

  用UE等支持二进制的编译器修改HelloWorld的魔数，在控制台运行

  ![image-20200206160841995](/images/image-20200206160841995.png)

* 准备：为static变量分配空间，设置默认值
  * <span style="color:red">**static变量在JDK7之前存储于instanceKclass末尾，从JDK7开始，存储于_java_mirror末尾**</span>
  * <span style="color:red">**static变量分配空间和赋值是两个步骤，分配空间在准备阶段完成，赋值在初始化阶段完成**</span>
  * <span style="color:red">**如果static变量是final的基本类型，那么编译阶段就确定了，赋值在准备阶段完成**</span>
  * <span style="color:red">**如果static变量是final的，但属于引用类型，那么赋值会在初始化阶段完成**</span>
* 解析：将常量池中的符号引用解析为直接引用

### 3.3.初始化

**<cinit>()v方法**

初始化即调用<cinit>()V，虚拟机会保证这个类的【构造方法】的线程安全

**发生的时机**

概括的说，类初始化是【**懒惰的**】

* main方法所在的类，总会被首先初始化
* 首次访问这个类的静态变量或静态方法时
* 子类初始化，如果父类还没有初始化，会引发

* 子类访问父类的静态变量，只会触发父类的初始化
* Class.forName
* new会导致初始化

不会导致初始化的情况：

* 访问类的static final静态常量(基本类型和字符串)不会触发初始化
* 类对象.class不会触发初始化

```java
/**
 * 类的初始化
 */
public class ClassLoad_02 {

    static {
        System.out.println("main init");
    }

    public static void main(String[] args) throws ClassNotFoundException {
        // 1.静态常量不会触发初始化
        System.out.println(B.b);
        // 2.类对象.class不会触发初始化
        System.out.println(B.class);
        // 3.创建该类的数组不会触发初始化
        System.out.println(new B[0]);
        // 4.不会初始化类B，但会加载A B
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        loader.loadClass("com.itcm.jvm.B");
        // 4.不会初始化类B，但会加载A B
        ClassLoader loader1 = Thread.currentThread().getContextClassLoader();
        Class.forName("com.itcm.jvm.B", false, loader1);


        // 1.首次访问类的静态变量或静态方法
        System.out.println(A.a);
        // 2.子类初始化，如果父类还没有初始化，会触发
        System.out.println(B.c);
        // 3.子类访问父类静态变量，只会触发父类初始化
        System.out.println(B.a);
        // 4.Class.forName()会初始B，并初始化A
        Class.forName("com.itcm.jvm.B");
    }
}

class B extends A {
    public static final int b = 10;
    public static boolean c = false;
    static {
        System.out.println("Class B init");
    }
}

class A {
    public static int a = 10;
    static {
        System.out.println("Class A init");
    }
}
```

### 3.4.类加载器

以JDK8为例：

| 名称                    | 加载哪些类            | 说明                        |
| ----------------------- | --------------------- | --------------------------- |
| Bootstrap ClassLoader   | JAVA_HOME/jre/lib     | 无法直接访问                |
| Extension ClassLoader   | JAVA_HOME/jre/lib/ext | 上级为Bootstrap，显示为null |
| Application ClassLoader | classpath             | 上级为Extension             |
| 自定义类加载器          | 自定义                | 上级为Application           |
|                         |                       |                             |

#### 3.4.1.启动类加载器

#### 3.4.2.扩展类加载器

```java
/**
 * 应用类加载器
 * 输出：
 * loader init
 * sun.misc.Launcher$AppClassLoader@18b4aac2
 */
public class ClassLoad_03 {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> aClass = Class.forName("com.itcm.jvm.Load");
        System.out.println(aClass.getClassLoader());
    }
}

class Load {
    static {
        System.out.println("loader init");
    }
}
```

#### 3.4.3.双亲委派模式

所谓的双亲委派，就是指调用类加载器的loadClass方法时，查找类的规则；

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded  查找该类是否被加载
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    // 有上级类加载器就委派给上级加载
                    c = parent.loadClass(name, false);
                } else {
                    // 如果没有上级Extension加载器，就委派给BoostrapClassLoader
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats 记录耗时
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

#### 3.4.4.线程上下文类加载器

比如我们用的jdbc驱动，com.mysql.jdbc.Driver;

#### 3.4.5.自定义类加载器























