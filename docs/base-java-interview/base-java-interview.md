# Java基础面试

## 1.数据结构

### 1.1.HashMap

#### **1.1.1.线程不安全的原因及表现？**

* 扩容时可能造成死循环，扩容时会造成死锁，形成环形链表；或者造成扩容大小不一致等问题；

* 多个线程put的时候，get的值可能不一致，put操作不是原子性；

* 删除键值对的时候，可能存在删除刚刚修改的位置信息；

#### **1.1.2.JDK1.7和JDK1.8比较**

* 底层结构：JDK7 数组+链表    JDK8 数组+链表+红黑树((<span style="color:red">**节点数>=8） &&桶的总个数（table的容量）>= 64)** )</span>
* 数组：JDK7 创建HashMap时，初始化数组table容量为16；JDK8：创建hashMap对象时，没有初始化table，仅仅只是初始化负载因子。当只有第一次添加时才会初始化table容量为16

* JDK7：table的类型为Entry   JDK8：table的类型为Node

### 1.2.CurrentHashMap

#### 1.2.1.JDK7实现

* ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表,同时又是一个ReentrantLock（Segment继承了ReentrantLock）；
* ConcurrentHashMap定位一个元素的过程需要进行两次Hash操作；
* 第一次Hash定位到Segment，第二次Hash定位到元素所在的链表的头部；
* <span style="color:red">**这一种结构的带来的副作用是Hash的过程要比普通的HashMap要长；**</span>

#### 1.2.2.JDK8实现

* <span style="color:red">**采用了数组+链表+红黑树的实现方式来设计，CAS+Synchronized保证线程安全；**</span>
* DK8中彻底放弃了Segment转而采用的是Node，其设计思想也不再是JDK1.7中的分段锁思想；
* Node：保存key，value及key的hash值的数据结构。其中value和next都用volatile修饰，保证并发的可见性；
* 锁的粒度：原来是对需要进行数据操作的Segment加锁，现调整为对每个数组元素加锁（Node）；