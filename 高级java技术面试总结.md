### 高级 Java 技术面试总结
- 主动复习策略：**只看标题，自己脑子里面进行回答**，不仅能调动积极性，也不至于昏昏欲睡，并且复习速度也更快。
- 多动笔写写，手写的比使用电脑记录内容，记忆程度更深；


### 0. 目录
1. [Java 基础](#1)
2. [JVM 相关](#2)
3. [Java 集合](#3)
4. [并发编程](#4)
5. [Spring](#5)
6. [MySQL](#6)
7. [Redis](#7)
8. [ElasticSearch](#8)
9. [消息队列(kafka)](#9)
10. [Dubbo](#10)
11. [设计模式](#11)
12. [算法](#12)
13. [领域驱动设计(DDD)](#13)
14. [系统设计相关](#14)

### 1. <span id="1">Java 基础</span>
1.1 八种基本数据类型：
  - `byte`，`short`，`int`，`long`，`float`，`double`，`boolean`，`char`

1.2 面向对象的三大特性：
  - 封装，继承，多态

1.3 多态的三个前提：
  - 两个类要有继承关系；
  - 子类要重写父类方法；
  - 父类引用指向子类；

1.4 接口与抽象类区别：
  - 一个类只能继承一个抽象类，但可以实现多个接口；
  - 抽象类可以提供成员方法的实现细节；接口中只能存在 `public abstract` 方法；
  - 抽象类中的成员变量可以是各种类型的；接口中的成员变量只能是 `public static final` 类型的；
  - 抽象类中可以有静态代码块和静态方法；接口中不能含有静态代码块及静态方法；

1.5 `final` 关键字
- 被 final 修饰的类不能被继承；
- 被 final 修饰的方法不能被覆盖重写；
- 被 final 修饰的成员变量，表示常量，只能被赋值一次，赋值后值不再改变，而且必须显式赋值；
- 被 final 修饰的局部变量(参数列表和方法内局部变量)，表示不可再更改；
- 安全发布，当创建一个对象时，使用 final 关键字能够使得另一个线程不会访问到处于“部分创建”的对象；

1.6 异常分类
- Error：`StackOverfowError`、`OutOfMemoryError`
- `java.lang.RuntimeException` 是 unchecked exception，子类包括：
  - `NullPointerException`、`ClassCastException`、`ArithmeticException`、`IndexOutOfBoundsException`；
- checked exception 是从 `java.lang.Exception` 衍生出来，非 `RuntimeException` 的子类异常，包括：
  - `IOException`、`ClassNotFoundException`、`SQLException`；
  - checked exception 在定义方法时，必须声明所有可能会抛出的 checked exception;
  - 在调用这个方法时，必须捕获它的 checked exception；

### 2. <span id="2">JVM 相关</span>
2.1 JVM 内存布局
- 按照线程是否共享来分类：

![JVM 内存布局](./images/JVM内存布局.png)

- 堆的默认空间分配
  - `-Xms`: memory start；
  - `-Xmx`: memory max；

![堆空间分配](./images/堆空间分配.png)

2.2 判断对象是否存活
- 可达性分析：从 GC Roots 为起点开始向下搜索，搜索所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是不可用的，可以被回收。

2.3 Java 中的引用
- 强引用（Strong Reference）：如果一个对象 GC Roots 可达，强引用不会被回收；
- 软引用（Soft Reference）：当 JVM 认为内存不足时，会清理软引用指向的对象；通常用来实现内存敏感的缓存；
- 弱引用（Weak Reference）：无论内存是否充足，只要发生 GC，都会回收跟弱引用关联的对象；
- 虚引用（Phantom Reference）：虚引用不会对对象生存时间构成影响，只是为了在这个对象被收集器回收时收到一个系统通知；

2.4 GC 算法
- 标记-清除算法(Mark-Sweep)：会产生大量不连续的内存碎片；
- 标记-复制算法(Coping)：每次是对整个半区进行内存回收，缺点：可用内存缩小到原来的一半，浪费一半的堆内存；
- 标记-压缩算法(Mark-Compact)：能够解决内存碎片化问题，但压缩算法的性能开销也不小；

2.5 垃圾回收器：
- 新生代：ParNew + 老年代：CMS
  - 启用 CMS 收集器：`-XX:+UseConcMarkSweepGC`
- CMS 运行过程：
  - 初始标记 -> 并发标记 -> 重新标记 -> 并发清除
  - 特点：并发收集，低停顿；
- G1 垃圾收集器
  - 将连续的堆内存划分为多个大小相等的 Region，可以通过参数`-XX:G1HeapRegionSize`设置，取值范围为 `1M ~ 32M`；
  - H 区：专门用来存储大对象；只要超过 Region 容量一半的对象即可判定为大对象；
  - 根据允许的收集时间，优先回收价值最大的 Region，避免 Full GC；
  - 使用 Remembered Set(记忆集) 避免全堆作为 GC Roots 扫描；
    - RSet 用于记录和维护 Region 之间的对象引用关系；用于记录其他 Region 中的对象引用本 Region 中对象的关系；
  - Collection Set(CSet)：记录了等待回收的 Region 集合；

2.6 JVM 调优
- JVM 调优的目的：尽量减少停顿时间，提高系统的吞吐量；
- 触发 Full GC 的情况：
  - 老年代空间不足，产生 Concurrent mode failure 或 promotion failed
    - Young GC 时，晋升老年代内存平均值大于老年代剩余空间；
    - Survivor 空间不足，Survivor 中对象还不足以晋升到老年代，从年轻代晋升到 Survivor 的对象大于 Survivor 剩余空间；
  - 元空间不足，扩容导致 STW 的 Full GC；
  - 有连续的大对象需要分配；
- 解决方案：
  - 老年代空间不足：
    - 增加新生代和老年代的内存；
    - 由于 Concurrent mode failure 发生后，CMS GC 会退回到 Serial Old GC，导致停顿时间延长；因此需要尽早执行 CMS GC，调低触发 CMS GC 执行的阈值：`-XX:CMSInitiatingOccupancyFraction = 68%`，默认值为 92%；
  - 元空间不足：
    - 配置：`-XX:MetaspaceSize=128 -XX:MaxMetaspaceSize=128`；
    - 元空间初始大小为 20.75M；

2.7 CPU 经常 100%，排查步骤：
- `top`：查看系统 CPU 的占用情况；
- `top -Hp [java 系统的 pid]`：查看该进程下各个线程 CPU 和内存占用情况；
- `jstack [进程号] | grep [线程号]`：查看线程栈情况
  - 如果结果中存在 `VM Thread`，则表示当前线程是垃圾回收的线程，也基本上可以确定，当前系统缓慢的原因主要是垃圾回收过于频繁，导致 GC 停顿时间较长；
- `jstat -gcutil [pid] 1000 10`：监测 GC 回收频率；

2.8 内存泄漏相关
- 定义：内存泄漏，是指程序内动态分配的堆内存由于某种原因程序未释放或者无法释放，导致系统内存浪费，程序运行速度变慢甚至系统崩溃等严重后果；
- 内存泄漏的表现：
  - 发生 OOM 错误；
  - 请求响应时间变长，因为频繁发生 Full GC，暂停其他业务线程(Stop The World)造成的；
- 方案：
  - 运行参数配置：`-XX:+HeapDumpOnOutOfMemoryError -XX:+HeapDumpPath=${指定 dump 文件的目录}`；
  - 使用 VisualVM 工具分析 dump 出的内存快照，定位主要是什么对象比较消耗内存，优化相关代码；

### 3. <span id="3">Java 集合</span>
3.1 String，StringBuilder 和 StringBuffer
- StringBuilder 可变，线程不安全；
- StringBuffer 可变，线程安全，内部方法使用了 synchronized 锁；
- String 不可变
  - String 类中真正存储字符的地方是 `private final char value[];`，被 final 修饰；
  - String 类被 final 修饰，不可继承；
  - String 源码中涉及到到 char 数组进行修改的操作全部都会重新创建一个 String 对象；

3.2 ArrayList 和 LinkedList
- ArrayList 基于动态数组，LinkedList 基于链表；
- 对于随机 index 访问，ArrayList 优于 LinkedList，ArrayList 可以直接通过数组下标找到元素；
- 新增和删除元素，LinkedList 优于 ArrayList，在新增或删除元素时，ArrayList 可能需要**扩容和复制**数组，而 LikedList 只需要修改指针即可；

3.3 CopyOnWriteArrayList
- 核心概念：读写分离，空间换时间；利用 CopyOnWrite（写时复制） 思想，在写时复制一份副本进行修改，修改完成后，再将新值赋给旧值；
- CopyOnWrite 的问题：
  - 内存占用问题：CopyOnWrite 因为复制副本，所以占用双倍内存，可能造成频繁的 Young GC 和 Full GC；
  - 数据一致性问题：CopyOnWrite 只保证了数据最终一致性，无法保证数据实时一致性；
- CopyOnWrite 适用场景：
  - 适合读多写少，且复制的对象不宜过大；
  - 添加、删除元素时，尽量使用 `addAll` 或 `removeAll` 批量添加(或删除)；

3.4 foreach 循环里面为什么不能进行元素的 `add/remove` 操作？
- 反编译 foreach 生成的 `.class` 文件，会发现增强 for 循环，底层依赖 `while` 循环和 `Iterator` 实现；
- 增强 for 循环，集合的遍历是通过 `Iterator` 进行的；
- 集合元素的 `add/remove` 操作，是通过 ArrayList 或 LinkedList 的 `add/remove` 方法，其源码中会调用 `modCount++`；
- 通过 `Iterator` 迭代时，每次调用 `next()` 方法，都会调用 `checkedForModification()` 方法，检查集合在遍历过程中是否被修改，如果 `modCount != expectedModCount`，会抛出 `ConcurrentModificationException`；
- **根本原因**：ArrayList 自带的 `add/remove` 方法不会去更新 Iterator 自身的 `expectedModCount` 值；
  - `fail-fast`，快速失败机制，当在迭代集合的过程中，集合元素发生变化时，就可能抛出 `ConcurrentModificationException`；
- 优化：`remove` 元素使用 Iterator 方式，使用 Iterator 的 `add/remove` 方法，除了调用对应集合的 `add/remove` 方法，还会修改自身的 `expectedModCount`；

<details>
<summary>反编译的 foreach 文件</summary>

```java
// 增强 for 循环
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    list.add("a");
    list.add("b");
    for(String item : list) {
        if ("b".equals(item)) {
            list.remove(item);
        }
    }
}

// IDEA 查看反编译后结果：
public static void main(String[] var0) {
    ArrayList var1 = new ArrayList();
    var1.add("a");
    var1.add("b");
    Iterator var2 = var1.iterator();

    while(var2.hasNext()) {
        // 此处调用 Iterator 中的 next 方法，会校验 modCount != expectedModCount
        String var3 = (String)var2.next();
        if ("b".equals(var3)) {
            // 此处调用 list 的 remove 方法，会修改 modCount
            var1.remove(var3);
        }
    }
}
```
</details>

3.5 HashMap
- 数据结构：数组 + 链表 + 红黑树；
- 扰动函数：优化散列效果，增加随机性，减少哈希碰撞；
  - `return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);`
- 初始化大小为 16；
- 负载因子：0.75，当阈值容量占了 3/4 时，就需要进行扩容，减少哈希碰撞；
- 扩容的算法：`e.hash & oldCap == 0`，判断当前节点是否需要移位，若为 0，不需要移动；若为 1，则需要移动至 (oldCap + 扩容容量)的位置；
- 链表转红黑树的条件：
  - 链表长度 >= 8，且存 key 值的数组桶容量大于 64；否则，只会扩容，不会树化；
- 不安全的原因：
  - Java 7：并发扩容时，采用**头插法**，造成链表逆序，容易出现环形链表，造成死循环；
  - Java 8：并发扩容时，采用**尾插法**，但是没有同步锁保护，可能造成数据被覆盖；

### 4. <span id="4">并发编程</span>
4.1 Thread 介绍
- `run()`：Thread 类继承了 `Runnable` 接口，重写了 `run()` 方法；`run()` 方法用于封装需要执行的任务；
- `start()`：用于线程的初始化，调度执行 `run()` 方法封装的任务；
- 创建 Thread 只有一种途径，借助构造方法，`new Thread` 对象；封装任务有两种方式：
  - 继承 Thread，重写 run 方法；
  - 实现 `Runnable` 接口，将实例对象传递给 Thread 构造函数；
  - 实现 `Callable` 接口，和 `FutureTask` 接口；

- `FutureTask` 介绍
  - 异步执行，可执行多次(通过 `runAndReset()` 方法)，也可以仅执行一次(执行 `run()` 方法）；
  - 可获取线程执行结果；

<details>
<summary>FutureTask 源码分析</summary>

```java
// FutureTask 的 run() 代码仅执行一次
public void run() {
    /**
     * FutureTask 的 run 方法仅执行一次的原因：
     *   1，state != NEW 表示任务正在被执行或已经执行完成，直接返回；
     *   2，若 state == NEW，则尝试 CAS 将当前线程设置为执行 `run()` 的线程，若设置失败，说明已经有其他线程在执行该任务了，当前线程退出；
     */
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                      null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}

// FutureTask 如何拿到线程执行的结果：
//   - 主要依赖 FutureTask 类内部的 Callable 属性；

// FutureTask 可能得执行过程：
//  顺利完成：NEW -> COMPLETING -> NORMAL
//  异常退出：NEW -> COMPLETING -> EXCEPTIONAL
//  被取消：  NEW -> CANCELLED
//  被中断：  NEW -> INTERRUPTING -> INTERRUPTED
```
</details>

4.2 Thread 状态
- Runnable 状态包含 Ready、Running 两种状态；
  - Ready：等待操作系统分配 CPU 时间片；
  - Running：正在占用 CPU 运行；

![线程状态转换图](./images/线程状态转换图.png)

4.3 `wait`、`sleep`、`join` 方法区别：
- `wait` 和 `sleep` 都会使线程进入阻塞状态，都是可中断方法，被中断后，会抛出 `InterruptedException`；
- `wait` 是 Object 方法，`sleep` 是 Thread 方法；
- `wait` 必须在同步方法(或同步代码块，`synchronized`)中使用，`sleep` 不需要；
- `wait` 会释放锁，`sleep` 不会释放锁；
- `join` 是 Thread 中的 synchronized 修饰的方法，里面调用了 `wait` 方法，让持有当前同步锁的线程进入等待状态，也就是主线程，当子线程执行完毕后，JVM 会调用 `lock.notifyAll(thread)` 方法，唤醒主线程继续执行；

4.4 `ThreadLocal`
- Thread 类有一个类型为 `ThreadLocal.ThreadLocalMap` 的实例变量 `threadLocals`，每个线程在往 `ThreadLocal` 里面放值时，都是往线程的 `ThreadLocalMap` 里面存，key 为 `ThreadLocal` 的一个弱引用；
- 使用 `ThreadLocal` 一定要记得执行 `new ThreadLocal<>().remove()` 方法，避免在发生 GC 后，弱引用 key 被回收，导致内存泄漏；
- **ThreadLocal 内存泄漏：** `ThreadLocalMap` 的生命周期和 Thread 一样长，当一个 ThreadLocal 的所有强引用都被移除后，但绑定了对应信息的线程还存在时，触发 GC 后，对应的 key 会被回收置为 null，而 value 还存在于内存中，但无法访问；

```java
// ThreadLocalMap 中的 Entry 类
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

![ThreadLocal 内存泄漏](./images/ThreadLocal内存泄漏.png)

4.5 内存模型(JMM，Java Memory Model)
- 主内存：所有线程共享；
- 工作内存：线程私有部分；

![内存模型](./images/内存模型.png)

4.6 线程安全问题，主要分为三类
- 原子性、可见性、有序性

- 原子性：多个操作作为一个整体，要么全部执行，要么不执行；
  - 锁机制：锁具有排他性，它能保证一个共享变量在任意一个时刻仅仅被一个线程访问；
  - 借助于处理器提供的 CAS 指令；
- 可见性：如果一个线程对某个共享变量进行更新之后，后续访问该变量的线程可以读到更新后的结果，则称为这个线程对该共享变量的更新对其他线程可见；
- 有序性：重排序是对内存访问操作的一种优化，它可以在不影响单线程程序正确的前提下，进行一定的调整，进而提高程序的性能；
  - `As-if-Serial Semantics`：貌似串行语义，从单线程的角度保障不会出现问题，但是对于多线程就可能出现问题；

4.7 `volatile`
- `volatile` 通过**内存屏障**来保证可见性、有序性以及自身修饰变量的原子性，但是不能保障代码段的原子性，是一种弱同步；
  - 当一个线程对 `volatile` 修饰的变量进行写操作时，JMM 会把最新值刷新到主内存；
  - 当一个线程对 `volatile` 修饰的变量进行读操作时，JMM 会把该线程对应的本地内存置为无效，从主内存中读取最新的值；

4.8 `synchronized`
- JVM 层面的悲观锁；
- 底层依赖监视器(Monitor)，监视器又依赖操作系统的互斥锁；
- 锁升级：无锁、偏向锁、轻量级锁、重量级锁、GC 标记；

![JVM 对象头](./images/JVM对象头.png)

- 锁的执行：`EntryList`、`WaitSet`；

![锁的执行](./images/锁的执行.png)

- 同步方法：`ACC_SYNCHRONIZED` 同步标识；
- 同步代码块：`monitorenter`、`monitorexist` 两条指令；
- 静态同步方法，锁对象为类；普通同步方法，锁对象为实例对象；
- 可重入特性：
  - 当前线程获得锁后，通过cas将`_owner`指向当前线程，若当前线程再次请求获得锁， `_owner`指向不变，执行`_recursions++`记录重入的次数，若尝试获得锁失败，则在`_EntryList`区域等待。

4.9 `synchronized` 和 `Lock` 对比
- `synchronized` 是 JVM 底层实现的，`Lock` 是 JDK 接口层面的；
- `synchronized` 是隐式的，`Lock` 是显式的，需要手动加锁和解锁；
- `synchronized` 无论如何都会释放，即使出现异常；`Lock` 需要自己保障正确释放；
- `synchronized` 是阻塞式获取锁，`Lock` 可以阻塞获取，可中断，还可以尝试获取，还可以设置超时等待获取；
- `synchronized` 无法判断锁的状态，`Lock` 可以判断锁的状态；
- `synchronized` 可重入，不可中断，非公平；`Lock` 可重入，可中断，可配置公平性(公平和非公平都可以)；
- 如果竞争不激烈，两者的性能都差不多的，可是 `synchronized` 的性能还在不断的优化；当竞争资源非常激烈时（即有大量线程同时竞争），此时`Lock`的性能要远远优于`synchronized`；

4.10 `AbstractQueuedSynchronizer`(AQS)
- AQS 基于 CLH 变体的虚拟双向队列；
- `volatile int state`：代表共享资源的状态，`state = 1` 代表当前对象锁已被占有，其他线程来加锁会失败；加锁失败的线程会被放入一个 FIFO 等待队列中，`UNSAFE.park()`方法阻塞当前线程；
- 通过 CAS 来保证 `state` 并发修改的安全性；
- `thread`：当前占有锁的线程；
- 锁的模式：独占锁(EXCLUSIVE)、共享锁(SHARED)；
- `waitStatus`：CANCELLED、SIGNAL、CONDITION、PROPAGATE、0(默认值)；

4.11 `CountDownLatch` 和 `CyclicBarrier`
- `CountDownLatch` 可以当作一个计数器来使用，比如主线程等待所有子线程都执行过某个时间点后，才能继续执行；
  - `CountDownLatch` 利用 AQS 的共享锁来进行线程的通知，`await` 方法等待 Latch 达到 0 或被中断，抛出异常后，才会被执行，即依次唤醒队列中的节点；
- `CyclicBarrier`：所有线程从同一时间点开始执行；
  - `CyclicBarrier` 利用 `ReentrantLock` 中的 `Condition` 来阻塞和通知线程；等到线程数增长到指定数量后，调用 `Condition.signalAll()` 唤醒所有线程；

4.12 锁的分类

a. 乐观锁与悲观锁
- 乐观锁认为自己在使用数据时，不会有别的线程修改数据，所以不会加锁，使用处理器提供的 CAS 指令来实现；
  - CAS 算法涉及到三个操作数：
    - 要读写的内存值 V；
    - 进行比较的值 E(预期值)；
    - 要写入的新值 N；
    - 当 `V == E` 时，用 N 更新 V；
  - CAS 算法的三个问题：
    - ABA 问题，可以使用时间戳或版本号解决；
    - 循环时间长开销大；
    - 只能保证一个共享变量的原子操作；
- 悲观锁认为自己在使用数据的时候，一定有别的线程来修改数据，因此在获取数据之前先加锁:
  - `synchronized` 或 `ReentrantLock`；

b. 公平锁与非公平锁
- 非公平锁：多个线程加锁时，直接尝试加锁，获取不到才会进入等待队列的队尾等待；
- 公平锁：是通过同步队列来实现多个线程按照申请锁的顺序来获取锁，从而实现公平性；
  - 公平锁的 `lock` 方法会调用 `hasQueuedPredecessors()` 方法，即判断当前线程是否位于同步队列中的第一个；
- 优缺点：
  - 公平锁不会产生线程饥饿，CPU 唤醒阻塞线程的开销会比非公平锁大，所以整体吞吐率比非公平锁低；
  - 非公平锁减少唤起线程的开销，整体的吞吐率高，但有可能产生线程饥饿，即等待队列中的线程一直获取不到锁；

c. 独占锁与共享锁
- 独占锁：是指该锁一次只能被一个线程所持有；`ReentrantLock` 是独占锁；
- 共享锁：是指该锁可被多个线程所持有；`ReentrantReadWriteLock` 是共享锁；

d. 可重入锁
- 可重入锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。`ReentrantLock`和`synchronized` 都是可重入锁，可重入锁的一个优点是可一定程度避免死锁。

e. 自旋锁
- 当前线程获取锁时，如果发现锁已经被其他线程占有，并不会马上阻塞自己，在不放弃 CPU 的情况下，多次尝试；自旋等待虽然避免了线程切换的开销，但是以浪费 CPU 为代价；

4.13 线程池

4.13.1 `Executors` 创建线程池的弊端：
- `FixedThreadPool` 和 `SingleThreadPool`：允许请求队列长度为 `Integer.MAX_VALUE`，可能会堆积大量请求，造成 OOM；
- `CachedThreadPool` 和 `ScheduledThreadPool`：允许创建的线程数量为 `Integer.MAX_VALUE`，可能会创建大量的线程，造成 OOM；

4.13.2 `ThreadPoolExecutor` 六大核心参数：
- `corePoolSize`：核心线程数；
- `maximumPoolSize`：最大线程数；
- `keepAliveTime`：空闲线程存活时间；
- `timeUnit`：时间单位；
- `workQueue`：存放待执行任务的队列；
- `handler`：当线程池线程数已满，且工作队列达到饱和，新提交的任务使用拒绝策略处理；
  - `AbortPolicy`：丢弃任务并抛异常，默认拒绝策略；
  - `DiscardPolicy`：丢弃任务但不抛异常；
  - `DiscardOldestPolicy`：丢弃队列最前面的任务，然后重新提交被拒绝的任务；
  - `CallerRunsPolicy`：由调用线程（提交任务的线程）处理该任务，从而降低新任务的流量；
- 动态调参：支持线程池参数动态调整，包括修改 `corePoolSize`、`maximumPoolSize`、`workQueue`；参数修改后及时生效；
- 增加线程池监控：包括线程池的任务执行情况、最大任务执行时间、平均任务执行时间等；

4.13.3 `Worker` 内部类
- `Worker` 内部类通过继承 AQS 来实现非公平，独占，不可重入锁；
- 通过 `Worker` 内部类可以掌握线程的运行状态，维护线程的生命周期；
  - `lock` 方法获取不到锁，表示当前线程正在执行任务中，不应该中断；
  - 线程池在执行 `shutdown` 方法或 `tryTerminate` 方法时，会调用 `interruptIdleWorkers` 方法来中断空闲的线程，`interruptIdleWorkers` 方法会使用 `tryLock` 方法来判断线程池中的线程是否是空闲状态；
  - Java 借助**中断机制**来停止一个线程：`public void interrupt();`；

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
  final Thread thread;

  Runnable firstTask;
}
```

### 5. <span id="5">Spring</span>
5.1 SpringMVC 流程图

![SpringMVC 流程图](./images/SpringMVC流程图.image)

5.1.1 拦截器和过滤器
- 拦截器：`org.aopalliance.intercept.Interceptor`，过滤器：`javax.servlet.Filter`；
- 实现原理不同：拦截器是基于 java 的反射（动态代理）机制，而过滤器是基于函数回调；
- 拦截器不依赖于 servlet 容器，过滤器则依赖 servlet 容器；
- 拦截器只能对 action 请求起作用，而过滤器则可以对几乎所有的请求起作用；
- 在 action 的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次；

5.2 `BeanFactory` 和 `ApplicationContext` 区别
- `BeanFactory` 是 Spring 容器最核心也是最基础的接口，用于管理 bean 的工厂，最核心的功能是加载 bean，即 `getBean` 方法，包含了各种 Bean 的定义，负责 Bean 的生命周期(读取配置文件，加载，实例化)，维护 bean 之间的依赖关系；
- `ApplicationContext` 接口继承了 `BeanFactory` 接口，除了提供 `BeanFactory` 所具有的功能外，还提供了更完整的框架功能：国际化、统一的资源文件访问方式(`ResourceLoader` 接口)、事件驱动机制(`ApplicationEventPublisher`接口)等；

5.3 `BeanFactory` 和 `FactoryBean` 的区别：
- `BeanFactory` 是 IOC 最基本的容器，负责生产和管理 bean，提供了一个 Spring IOC 容器规范，`DefaultListableBeanFactory`、`XmlBeanFactory`、`ApplicationContext` 等具体的容器都实现了 `BeanFactory`；
- `FactoryBean`：工厂 bean，实现该接口的类可以自定义想要创建的 bean 实例，`FactoryBean#getObject`返回一个代理类，在代理类中可以实现自定义逻辑，如自定义的监控、限流等等；

5.4 动态代理
- JDK 动态代理，只能基于接口进行代理，通过反编译，可以发现代理类继承自 Proxy;
- Cglib 代理：基于 ASM 字节码，在运行时对字节码进行修改和动态生成，通过继承的方式进行代理，无论对象有没有实现接口，都可以进行代理；

5.5 控制反转(Inversion of Control, IOC)
- 由 Spring 来负责控制对象的生命周期，`@ComponentScan` 定义扫描包路径，将 `@Component`、`@Controller`、`@Service`、`@Repository` 等注解的类加载到 Spring 容器；
  - `@Component` 是通用注解，其他三个注解是这个注解的衍生注解；
  - `@Controller` 是 SpringMVC 注解，用于控制层；
  - `@Service` 用于业务逻辑层；
  - `@Repository` 用于持久层，标注 DAO 类；

- `@Configuration` 和 `@Component` 区别：
  - `@Configuration` 本质上还是 `@Component`，`@Configuration` 中所有带 `@Bean` 注解的方法都会被动态代理，调用该方法返回的都是同一个实例对象；
  - `@Component` 修饰的类，每次都会创建一个新的对象返回；

5.6 依赖注入(Dependency Injection, DI)
- 依赖注入，常见 3 种方式：
  - 属性注入
  - Setter 方法注入
  - 构造方法注入

- `@Autowired` 和 `@Resource` 用于维护 bean 之间的依赖关系；
  - 相同点：
    - `@Autowired` 和 `@Resource` 都是作为 bean 对象注入的时候使用；
    - 两者都可以声明在字段和 setter 方法上；
  - 不同点：
    - `@Autowired` 是 spring 提供的，`@Resource` 是 J2EE 提供的；
    - `@Autowired` 注入的对象需要在 IOC 容器种存在，否则需要加上属性 `required = false`，表示忽略当前要注入的 bean；
    - 注入方式：
      - `@Autowired` 默认是 byType 方式注入，可以配合 `@Qualifier` 注解来显式指定 name 的值；
      - `@Resource` 默认是 byName 方式注入，如果没有匹配，则通过 byType 注入；`@Resource` 还有两个重要的属性：name 和 type，用来显示指定 byName 和 byType 方式注入；

5.7 Bean 生命周期
- 概述

![bean 生命周期概述](./images/bean生命周期概述.png)

- 整体描述

![bean 生命周期整体描述](./images/bean生命周期整体描述.png)

5.7.1 `BeanDefinition`
- `BeanDefinition` 接口用于描述 bean 的元信息，包含 bean 的类信息(全限定类名 beanClassName)、属性（作用域、描述信息）、依赖关系等；**主要目的**是允许 `BeanFactoryPostProcessor` 拦截修改属性值和其他 bean 的元数据；
  - `AbstractBeanDefinition`：抽象类，默认实现了 `BeanDefinition` 中的绝大部分方法；
  - `GenericBeanDefinition`：相比 `AbstractBeanDefinition`，新增 `parentName` 属性，可以灵活设置 parent bean definition；
  - `ScannedGenericBeanDefinition`：通过 `@Component`、`@Controller`、`@Service` 等方式注解的类，会以 `ScannedGenericBeanDefinition` 的形式存在；
  - `AnnotatedGenericBeanDefinition`：借助于 `@Import` 导入的 bean；
- `BeanDefinitionRegistry` 是维护 `BeanDefinition` 的注册中心，它内部存放了 IOC 容器中的 bean 定义信息。它的实现类有 `DefaultListableBeanFactory`；

5.7.2 spring 中的常见扩展点
- `BeanFactoryPostProcessor`：在 `BeanDefinition` 加载完成之后，未实例化之前，定制化修改 `BeanDefinition`；
- `BeanPostProcessor`：在 bean 的初始化阶段前后添加自定义处理逻辑，例如 AOP 通过 `AbstractAutoProxyCreator#postProcessAfterInitialization` 方法生产代理 bean 等；

5.7.3 spring 与 Mybatis 整合
- `MapperScannerConfigurer` 扫描注册 basePackage 包下的所有 bean，并将对应的接口类型改造为 `MapperFactoryBean`；
  - `MapperFactoryBean` 继承了 `SqlSessionDaoSupport` 类，`SqlSessionDaoSupport` 类继承 `DaoSupport` 抽象类，`DaoSupport` 抽象类实现了 `InitializingBean` 接口；
  - `MapperFactoryBean` 的出现是为了代替手工使用 `SqlSessionDaoSupport` 或 `SqlSessionTemplate` 编写数据访问对象(DAO)的代码，使用动态代理实现；
- XML 中的 SQL 会被解析并保存到本地缓存中，key 是 SQL 的 namespace + id， value 是 SQL 的封装；
- 当我们调用 DAO 接口时，会走到代理类中，通过接口的全路径名，从步骤 2 的缓存中找到对应的 SQL，然后执行并返回结果；

5.7.2 三级缓存与循环依赖
- 循环依赖分类：
  - 构造器循环依赖：
    - spring 无法解决此类依赖，因为创建 bean 需要使用构造器，当构造函数出现循环依赖时，我们无法创建“不完整”的 bean 实例；
    - 会抛出 `BeanCurrentlyInCreationException`；
  - 赋值属性循环依赖：
    - spring 只支持 bean 在 **单例模式（singleton)** 下的循环依赖；其他模式下的循环依赖，会抛出 `BeanCurrentlyInCreationException`；
- 解决循环依赖的方式：提前暴露创建中的 bean 实例；
  - 一级缓存 `singletonObjects`：存储所有创建完成的单例 bean；
  - 二级缓存 `earlySingletonObjects`：完成实例化，但未进行属性注入及初始化的对象，即提前暴露的单例缓存，`ObjectFactory` 返回的 `bean`；
  - 三级缓存 `singletonFactories`：生产单例的工厂缓存`ObjectFactory`；
- 使用两级缓存是否可以？
  - 不可以；
  - Bean 的生命周期：`实例化 -> 属性注入 -> 初始化`；
  - 若不使用三级缓存，在对象 A 实例化后，就需要马上为 A 创建代理，然后放入到二级缓存中去，这样，违背了 Spring 中 AOP 与 Bean 的生命周期相结合的设计原则；
  - AOP 与 Bean 的生命周期结合，是通过 `AnnotationAwareAspectJAutoProxyCreator` 这个后置处理器完成的，调用其中的 `postProcessAfterInitialization` 方法，对初始化后的 Bean 完成动态代理；

![三级缓存对象创建过程](./images/三级缓存对象创建过程.png)

5.7.3 spring 事务的传播行为
- 支持当前事务：
  - **REQUIRED**：默认，若当前没有事务，则建立一个新事务；若存在事务，直接加入；
  - **SUPPORTS**：若当前有事务，直接加入；若没有事务，以非事务的方式执行；
  - **MANDATORY**：若当前有事务，直接加入；若没有事务，抛异常；
- 不支持当前事务：
  - **REQUIRED_NEW**：新建事务，若当前存在事务，把当前事务挂起；
  - **SUPPORTED**：若当前没有事务，则以非事务的方式操作；若当前存在事务，把当前事务挂起；
  - **NEVER**：若当前没有事务，则以非事务的方式操作；若当前存在事务，抛异常；
- 嵌套事务：
  - **NESTED**：嵌套事务呈现父子事务的概念，父子之间是有关联的，核心思想就是子事务不会独立提交，而是取决于父事务，当父事务提交，子事务才会提交；若父事务回滚，则子事务也回滚；
  - 若当前存在事务，则会在内部开启一个新事务，作为嵌套事务存在；
  - 若当前无事务，则开启新事务，类似`REQUIRED`；
- `REQUIRED_NEW`示例：
  - `aMethod` 调用 `bMethod`，`aMethod` 用 `PROPAGATION_REQUIRED` 修饰，`bMethod` 用 `REQUIRED_NEW` 修饰；
    - `aMethod` 异常，`bMethod` 不会回滚，因为 `bMethod` 开启了独立的事务；
    - 若 `bMethod` 抛出了未捕获异常，且这个异常满足事务回滚规则，`aMethod` 也会回滚，因为这个异常被 `aMethod` 的事务管理机制监测到了；

5.7.4 Spring 事务的实现原理
- Spring 事务的底层实现主要使用的技术：`AOP(动态代理） + ThreadLocal + try/catch`；
  - 动态代理：基本所有要进行逻辑增强的地方都会用到动态代理，AOP 底层也是动态代理实现的；
  - `ThreadLocal`：主要用于线程间的资源隔离，以此实现不同线程可以使用不同的数据源、隔离级别等；
  - `try/catch`：最终是执行 `commit` 还是 `rollback`，是根据业务逻辑处理是否抛出异常来决定；

5.8 SpringBoot 自动装配原理
- SpringBoot 启动时，会执行 `SpringApplication.run()`，`run()` 方法会刷新容器，刷新容器时，会解析启动类上的注解 `@SpringBootApplication`，这是个复合注解，其中有三个比较重要的注解 `@SpringBootConfiguration`、`@ComponentScan`、`@EnableAutoConfiguration`：
  - `@SpringBootConfiguration` 底层是 `@Configuration` 注解，通过 `@Configuration` 和 `@Bean` 结合，将 Bean 注册到 Spring IOC 容器；
  - `@ComponentScan`  扫描注解，默认是扫描当前类下的 package，将 `@Component`、`@Controller`、`@Service`、`@Repository` 等注解的类加载到 IOC 容器中；
  - `@EnableAutoConfiguration` 开启自动配置，是一个复合注解；
    - `@AutoConfigurationPackage`：自动配置包；
    - `@Import(AutoConfigurationImportSelector.class)`：会扫描所有 jar 路径下的 `META-INF/spring.factories`，将其文件包装成 `Properties` 对象，从 `Properties` 对象获取 key 值为 `EnableAutoConfiguration` 所对应的数据，加载到 IOC 容器，根据配置类上的条件注解 `@ConditionalOnXXX` 来判断是否将这些配置类在容器中进行实例化；

5.9 类加载机制
- **双亲委派模型**：一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时，才尝试自己加载；
  - 启动类加载器(Bootstrap ClassLoader) 是由 C++ 实现的，并不是继承自 `java.lang.ClassLoader`，没有对应的 Java 对象，举例来说，`java.lang.String` 是由启动类加载器加载的，而 `String.class.getClassLoader()` 就会返回 null；
  - 扩展类加载器(Extension ClassLoader) 和应用程序类加载器(Application ClassLoader) 是 `sun.misc.Launcher` 的内部类，均继承自 `java.net.URLClassLoader`，`URLClassLoader` 继承自抽象类 `java.lang.ClassLoader`：
    - 每个 `ClassLoader` 都持有一个 `parent` 字段，指向父加载器，这个 `parent` 字段从 `java.lang.ClassLoader` 继承而来；
- `java.lang.ClassLoader` 中的三个方法：
  - `defineClass`：调用 native 方法把 Java 类的字节码解析成一个 Class 对象；
  - `findClass`：把来自文件系统或网络的 `.class` 文件读取到内存，得到字节码数组，然后调用 `defineClass` 方法得到 Class 对象；
  - `loadClass`：默认按照**双亲委派模型**来加载类，具体加载过程：
    - 调用 `findLoadedClass(name)` 检查类是否已经加载过；若没有，则继续；
    - 若 `parent` 属性值不为 null，根据双亲委派模型，调用 `parent.loadClass(name, false)`，优先从 parent 中执行 loadClass；
    - 若 `parent` 属性值为 null，则调用 `findBootStrapClassOrNull(name)` 判断是否在 `BootStrapClassLoader` 中加载过；
    - 如果类仍未找到，则执行 `findClass` 查找类，`findClass` 由自定义的 `ClassLoader` 实现；
- 自定义 ClassLoader，是通过继承 `java.lang.ClassLoader` 抽象类，重写以下方法：
  - 如果希望**遵循**双亲委派模型，重写 `findClass()` 方法；
  - 如果希望**打破**双亲委派模型，重写 `loadClass()` 方法；

### 6. <span id="6">MySql</span>
6.1 SQL 语句执行流程
- MySQL 分为 Server 层和存储引擎层两部分：
  - Server 层包括连接器、查询引擎、分析器、优化器、执行器等；
  - 存储引擎层负责数据的存储和提取；

![mysql的逻辑架构图](./mysql/images/mysql的逻辑架构图.webp)

- 连接器：管理连接，权限验证；
  - 负责跟客户端建立连接、获取权限、维持和管理连接；
- 查询缓存
- 分析器：SQL 的词法分析，语法分析；
  - 根据语法规则，判断输入的 SQL 语句是否满足 MySQL 语法规则；
- 优化器：生成 SQL 执行计划，索引选择；
  - 数据表里面有多个索引的时候，优化器决定使用哪个索引；或者在一个语句有多表关联(join)的时候，决定各个表的连接顺序；
- 执行器：操作存储引擎，返回查询结果；
  - 判断对指定表有没有执行查询权限，若有权限，则调用存储引擎的接口，执行查询；

6.2 存储引擎：InnoDB 和 MyISAM
- InnoDB 支持事务，MyISAM 不支持；
- InnoDB 支持外键，MyISAM 不支持；
- InnoDB 使用聚集索引，索引结构为 B+Tree，叶子节点保存了主键和数据记录；MyISAM 使用非聚集索引，索引和数据文件是分离的，索引中保存的是数据文件的指针；
- InnoDB 支持表、行级锁，MyISAM 支持表级锁；

6.3 InnoDB Buffer Pool(InnoDB 缓冲池)
- `Buffer Pool` 用于缓存表数据和索引数据，避免每次访问都进行磁盘 IO，起到加速访问的作用；
- 预读机制：
  - 根据“局部性原理”，使用一些数据，大概率会使用附近的数据；磁盘读写，并不是按需读取，而是按页读取，一次至少读取一页数据（一般是 4KB）；
- `Buffer Pool` 采用变种 LRU 算法(最近最少使用)，将 LRU 队列分成新生代(new sublist)和老生代(old sublist)两个区域，新生代用来存热点数据页，老生代用来存使用频率较低的数据页，默认比例为 63:37；
  - 将缓冲池分为老生代和新生代，进入缓冲池的页，优先进入老生代，页被访问且在老年代停留时间超过配置的阈值，才进入新生代，以解决预读失效和缓存污染的问题；
  - 预读失效：由于预读(Read-Ahead)，提前把页放入缓冲池，但最终 MySQL 并没有从页中读取数据，称为预读失效；
  - 缓冲池污染：当某一个 SQL 语句，要批量扫描大量数据时，可能导致把缓冲池的所有页都替换出去，导致大量热数据被换出，MySQL 性能急剧下降，这种情况属于缓冲池污染；
- 参数：
  - `innodb_buffer_pool_size`：配置缓冲池的大小；
  - `innodb_old_blocks_pct`：老生代占整个 LRU 链长度的比例，默认是 37；
  - `innodb_old_blocks_time`：老生代停留时间窗口，单位是毫秒，默认是 1000；

6.4 Change Buffer
- 写缓冲是一种应用在**非唯一普通索引页**(non-unique secondary index page)不在缓冲池中，对页进行了写操作，并不会立刻将磁盘页加载到缓冲池，而仅仅记录缓冲变更(buffer changes)，等未来数据被读取时，再将数据合并(merge)恢复到缓冲池中的技术。写缓冲的目的是降低写操作的磁盘 IO，提升数据库性能；
  - 如果索引设置了唯一属性，在进行修改操作时，InnoDB 必须进行唯一性检查；
- 写缓冲场景：
  - 数据库大部分是非唯一索引；
  - 业务是写多读少，或者不是写后立刻读取；
    - 如果写后，立刻读取它；因为读取时，本来要进行页读取，相应页面就要入缓冲池，写缓冲增加了复杂度；
- 参数：
  - `innodb_change_buffer_max_size`：配置写缓冲的大小，占整个缓冲池的比例，默认值是 25%；
  - `innodb_change_buffering`：配置哪些写操作启用写缓冲，可以设置成 `all/none/inserts/deletes`等；

6.5 redolog(重做日志)

![redo log存储状态](./images/redolog存储状态.webp)

- redo log 可能存在三种状态：
  - 存在 redo log buffer 中，物理上是在 MySQL 进程内存中，就是图中的红色部分；
  - 写到磁盘(write)，但是没有持久化(fsync)，物理上是在文件系统的 page cache 里面，也就是图中的黄色部分；
  - 持久化到磁盘，对应的是 hard disk，也就是图中的绿色部分。
- 参数：`innodb_flush_log_at_trx_commit`
  - 为 0，表示延迟写，每次事务提交时，都只是把 redo log 留在 redo log buffer 中；
  - 为 1，表示实时写，实时刷，每次事务提交时，都将 redo log 直接持久化到磁盘；
  - 为 2，表示实时写，延迟刷，每次事务提交时，只是把 redo log 写到 page cache；
  - InnoDB 有一个后台线程，每隔 1 秒，就会把 redo log buffer 中的日志，调用 write 写到文件系统的 page cache，然后调用 fsync 持久化到磁盘；

![redolog示例](./images/redolog示例.webp)

- InnoDB 的 redo log 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1G；
- `write pos`：是当前记录的位置，一边写一边后移，写到第 3 号文件末尾后就回到 0 号文件开头；
- `checkpoint`：是当前要擦除的位置，擦除记录前要把记录更新到数据文件；
- `write pos` 和 `checkpoint` 之间还空着的部分，可以用来记录新的操作；

6.6 binlog(归档日志)

<details>
<summary>实验准备</summary>

```sql
-- 建表和插入语句
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `t_modified` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `a` (`a`),
  KEY `t_modified`(`t_modified`)
) ENGINE=InnoDB;

insert into t values(1,1,'2018-11-13');
insert into t values(2,2,'2018-11-12');
insert into t values(3,3,'2018-11-11');
insert into t values(4,4,'2018-11-10');
insert into t values(5,5,'2018-11-09');

-- 删除语句
delete from t /*comment*/  where a>=4 and t_modified<='2018-11-10' limit 1;
```
</details>

6.6.1 statement 格式
- 当`binlog_format=statement`时，binlog 里面记录的就是 SQL 语句原文；
- 查看 binlog 中的内容：`show binlog events in 'mysql-bin.000092';`

![statement格式 binlog 示例](./images/statement格式binlog示例.png)

- 说明：
  - 第二行是一个 BEGIN，跟第四行的 COMMIT 对应，表示中间是一个事务；
  - 第三行就是真实执行的语句。`use lottery` 命令，是 MySQL 根据当前要操作的表所在的数据库，自行添加的。这样做可以保证日志传到备库去执行的时候，不论当前的工作线程在哪个库里，都能够正确的执行；
  - 最后一行是一个 COMMIT。`xid=7`，事务 id；
- 存在问题：
  - 由于 `delete` 带 `limit`，很可能会出现主备数据不一致的情况，比如：
  - 如果 `delete` 语句使用的是索引 a，那么会根据索引 a 找到第一个满足条件的行，即 `a=4`这一行；
  - 如果使用的是索引`t_modified`，那么删除的是 `t_modified='2018-11-09'`，也就是 `a=5`这一行；
  - 而 `binlog_format=statement` 格式下，binlog 里面记录的是语句原文，可能会出现这样一种情况：在主库执行这条 SQL 语句的时候，用的是索引 a；而在备库执行这条语句的时候，却使用了索引 `t_modified`，导致主备数据不一致； 


6.6.2 row 格式

![row格式 binlog 示例](./images/row格式binlog示例.png)

- 与 statement 格式相比，前后的 BEGIN 和 COMMIT 是一样的。但是，row 格式的 binlog 里没有 SQL 语句原文，而是替换成了两个 event：`Table_map` 和 `Delete_rows`：
  - `Table_map`：用于说明接下来要操作的表是 `lottery` 库的表 `t`;
  - `Delete_rows`：用于定义删除的行为；
- 借助 `mysqlbinlog` 工具，解析和查看 binlog 中的详细信息；上图第 2 列，显示这个事务的binlog 是从 `154` 开始的：
  - `mysqlbinlog --no-defaults -vv mysql-bin.000093 --start-position=154`

![row格式 binlog 详细信息](./images/row格式binlog的详细信息.png)

- `mysqlbinlog -vv`：是为了把内容都解析出来，所以从结果里面可以看到各个字段的值（比如，@1=5、@2=5 这些值）；
- `binlog_row_image` 的默认配置是 FULL，因此 Delete_event 里面，包含了删掉行的所有字段的值。如果把 `binlog_row_image` 设置为 MINIMAL，则只会记录必要的信息，在这个例子里，就是只会记录 id=4 这个信息；
- 当 `binlog_format` 使用 row 格式的时候，binlog 里面记录了真实删除行的主键 id，这样 binlog 传到备库去的时候，就肯定会删除 id=5 的行，不会有主备删除不同行的问题；

6.6.3 mixed 格式
- `statement` 格式的 binlog 可能会导致主备不一致，所以要使用 `row` 格式；
- `row` 格式的 binlog 体积可能会非常大；比如 delete 删掉 10 万行数据，用 `statement`格式， binlog 中只会记录一个 SQL 语句，占用几十个字节的空间；而 `row` 格式，这 10 万条数据都会写到 binlog 中。这样做，不仅会占用更大的空间，同时写 binlog 也要耗费 IO 资源，影响执行速度；
- `mixed`格式：Mysql 根据 SQL 语句是否可能引起主备不一致，如果有可能，就用 `row` 格式，否则就用 `statement` 格式；
- **重要：**
  - 现在越来越多的场景要求把 binlog 的格式设置为 **row**；好处之一：恢复数据；
  - 分别从`delete`、`insert`和`update`这三种 SQL 语句，来看看数据恢复问题：
  - 在 row 格式下，binlog 会记录被删掉的整行信息。所以，执行完 delete 语句后，发现删错数据了，可以把 binlog 中记录的 delete 语句转成 insert，把被错删的数据插入回去就可以恢复了；
  - row 格式下，binlog 会记录 insert 语句的所有字段信息；insert 错了，可以直接把 insert 语句转成 delete 语句，删除掉被误插入的数据即可；
  - 如果执行的是 update 语句，binlog 里面会记录**修改前整行的数据和修改后的整行数据**。所以，如果误执行了 update 语句，只需要把这个 event 前后的两行信息对调一下，再去数据库里面执行，就能恢复这个更新操作了；

6.7 redolog 和 binlog 区别
- redolog 是 InnoDB 引擎特有的，binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用；
- redolog 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1”；
- redolog 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志；

6.8 两阶段提交：
- `写 redolog -> 事务状态置为 prepare -> 写 binlog -> 提交事务 -> 修改 redolog 事务状态为 commit`；

![两阶段提交示意图](./images/两阶段提交示意图.webp)

- 在两阶段提交的不同时刻，MySQL 异常重启会出现什么现象？
  - 时刻 A，写入 redo log 处于 prepare 阶段之后，写 binlog 之前，发生了崩溃(crash)，由于此时 binlog 还没写，redo log 也还没提交，所以崩溃恢复的时候，这个事务会回滚；
  - 时刻 B，binlog 写完，redo log 还没 commit 前发生 crash，如何崩溃恢复？
  - 崩溃恢复判断规则：
    - 如果 redo log 里面的事务是完整的，即已经有了 commit 标识，则直接提交；
    - 如果 redo log 里面的事务只有完整的 prepare，则应判断对应的事务 binlog 是否存在并完整：
      - 如果是，则提交事务；(时刻 B 对应的情况)
      - 否则，回滚事务；

6.9 索引

6.9.1 B+ 树和 B 树的优势
- B+ 树的所有数据都在叶子节点，非叶子节点存储的是指向其他节点的指针；而 B 树的非叶子节点也保存具体的数据；同样大小的情况下，B+ 树可以存储更多的关键字，B+ 树比 B 树更加矮胖，IO 次数少；
- B+ 树叶子结点使用双向链表前后关联，更加方便范围查询，即由于 B+ 树所有的数据都在叶子节点，并且节点之间由指针连接，在查找大于（或小于）某个关键字时，只需要找到该关键字，然后沿着链表遍历即可；
- B+ 树更有利于对数据扫描，避免 B 树的回溯扫描；

6.9.2 索引设计原则
- 索引不是越多越好，维护索引需要时间和空间；
- 频繁更新的数据不宜建索引；
- 频繁 `group by`、`order by` 的列建议生成索引，可以大幅提高分组和排序效率；
- 在区分度高的字段上建立索引，区分度太低，无法有效的利用索引，可能需要扫描所有的数据页，此时和不使用索引差不多；
- 查询记录的使用，少使用 `select *`，尽量使用覆盖索引，可以减少回表操作，提升效率；

6.9.3 索引失效的场景
- 模糊搜索，左模糊或全模糊都会导致索引失效，如`like %a`或`like %a%`，右模糊可以利用索引；
- 隐式类型转换，`select * from customer_info where name = 李四;` 中 name 是字符串类型，但是没有加引号，查询时，MySQL 会进行隐式转换，导致索引失效；
- 索引字段使用函数或运算符操作，导致索引失效；
- `or` 条件索引失效，条件中如果有 or，只要其中一个条件没有索引，其他字段有索引也不会用到；即 or 前后存在非索引列时，索引失效；`select * from stu where name='Tom' or age = 14`，name 和 age 均为普通索引，or 是可以使用 `index_merge` 合并索引；
- 不符合联合索引的最左匹配原则：(A, B, C) 的联合索引，只 where 了单列 B 或 C 或多列(B, C)；
  - `a = 1 and b > 0 and c = 1`，c 不走索引，多字段索引的情况下，mysql 会一直向右匹配直到遇到范围查询(>，<，between，like)，就停止匹配；

6.9.4 索引名词
- InnoDB 主键选择：
  - InnoDB 推荐使用自增 ID 作为主键，自增 ID 可以保证每次插入时，B+ 树的索引是向右扩展的，可以避免 B+ 树的频繁合并和分裂(对比使用 UUID)，如果使用字符串主键或随机主键，会使得数据随机插入，效率比较差；
- InnoDB 有两种索引：主键索引（聚集索引）、辅助索引（非聚集索引、二级索引）
  - 主键索引：每个表中只有一个主键索引，叶子结点同时保存了主键的值和数据行记录；
  - 辅助索引：叶子结点保存了索引字段的值以及主键的值；
- 回表：先通过数据库普通索引扫描出数据所在的行，再通过行主键 ID 取出索引中未提供的数据，即基于非主键索引的查询需要多扫描一棵索引树；
- 覆盖索引：如果一个索引包含（或者说覆盖）所有需要查询的字段值，我们就称之为覆盖索引；
- 联合索引：相对单列索引，联合索引是用多个列组合构建的索引；
- 索引下推：MySQL 5.6 引入了索引下推优化，可以在索引遍历过程中，对索引中包含的字段先做判断，过滤掉不符合条件的记录，减少回表数据量；

6.9.5 Explain 关键字
- `key`：查看有没有使用索引，key 为 null，说明没有用到索引；
- `ken_len`：查看索引使用是否充分；
- `type`：查看索引类型，all 表示全表扫描，即需要进行优化；
  - `ref`：出现于 where 操作符为 "="且 where 字段为非唯一索引的单表或联表查询；
  - `eq_ref`：出现于 where 操作符为 "="且 where 字段为唯一索引的联表查询；
  - `range`：部分索引扫描，当查询为区间查询，且查询字段为索引字段时，这时会根据 where 条件对索引进行部分扫描；
- `extra`：查看附加信息；
  - `using index`：代表使用覆盖索引，不用回表；
  - `using filesort`：代表 order by 字段不是索引，mysql 无法利用索引进行排序，需要优化；
  - `using temporary`：创建了临时表来保存中间结果，需要优化；
- `rows`：mysql 估算要查找到结果集需要扫描读取的数据行数，根据 rows 可以直观看出优化效果；

6.9.6 快速查询 1000 万至 1000 万 100 条数据
- 采用子查询的方式：
  - `select * from single_info where id >= (select id from single_info limit 10000000, 1) limit 100;`

6.10 MySQL 事务

6.10.1 事务隔离级别
- ACID：原子性(Atomicity)、一致性(Consistency)、隔离性(Isolation)、持久性(Durability)；

6.10.2 事务操作可能会出现的数据问题
- 脏读：A 事务读到了 B 事务未提交的数据，若 B 事务回滚，则 A 事务出错；
- 不可重复读：重点是数据发生了修改，两次读取，数据的值不一样；
- 幻读：专指新插入的行，两次查询，发现新增了一条未处理的记录；

6.10.3 事务隔离级别
- 读未提交、读已提交、可重复度、串行化
- InnoDB 默认为可重复读，在可重复读隔离级别下，普通查询是快照读，是不会看到别的事务插入的数据；幻读在当前读下才会出现，要用间隙锁解决此问题；

6.10.4 快照读
- 快照读：不加锁的非阻塞读，为了提高并发性能，基于多版本并发控制实现(MVCC)；
- MVCC 实现原理：
  - 在 InnoDB 中，每行记录实际上都包含了两个隐藏字段：事务 id(trx_id)和回滚指针(roll_pointer)；
  - 回滚指针，指向这条记录的上一个版本，事务对一条记录的修改，会导致该记录的 undo log 成为一条记录版本线性表(链表)，undo log 链首就是最新的旧记录，链尾就是最早的旧记录；
- ReadView(读视图)：事务进行快照读操作的时候会产生读视图，主要包含以下 4 个内容：
  - `creator_trx_id`：表示生成该 ReadView 事务的事务 id；
  - `m_ids`：表示在生成 ReadView 时，当前系统中活跃的读写事务的事务 id 列表；
  - `min_trx_id`：在生成 ReadView 时，当前系统中活跃的读写事务中最小的事务 id，即 m_ids 中的最小值；
  - `max_trx_id`：表示生成 ReadView 时，系统中应该分配给下一个事务的 id 值；

6.10.5 当前读
- `select ... lock in share mode(共享锁)`、`select ... for update`、`update`、`insert`、`delete` 这些操作都是当前读，即读取记录的最新版本，会对读取的记录进行加锁；

![当前读示例图](./images/forupdate查询.png)

- `select ... for update;`
  - 如果 `for update` 没有命中索引，会锁表；
  - 如果数据不存在，不会锁表；
  - 事务 A 提交后，事务 B 可以读取到事务 A 新增的数据，`select ... for update;` 为当前读；

6.10.6 MySQL 中的锁
- 表锁和行锁
  - 表级锁：开销小、加锁快、加锁粒度大、发生锁冲突概率高、支持的并发度低；
  - 行级锁：开销大、加锁慢、加锁粒度小、发生锁冲突概率低、支持的并发度高；
- 意向锁
  - 意向共享锁(IS Lock，Intention Shared Lock)；
  - 意向排他锁(IX Lock，Intention Exclusive Lock)；
  - 表中若存在意向锁，表明某个事务正在或即将锁定表中的数据行，是为了处理行锁和表锁之间的矛盾；
- 行锁(Record Lock)、间隙锁(Gap Lock)、临键锁(Next-Key Lock)
  - 行锁只能锁住行，如果往记录之间的间隙插入数据，就无法解决，因此 MySQL 引入间隙锁，间隙锁是左右开区间，间隙锁之间不会冲突；
  - 间隙锁和行锁，合称 Next-Key Lock，区间为**左开右闭**；
- 加锁原则：
  - 加锁的基本单位是 `Next-Key Lock`，左开右闭；
  - 查找过程中访问到的对象才会加锁；
  - 索引上的等值查询，给唯一索引加锁的时候，`Next-Key Lock` 会退化为行锁；
  - 索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，`Next-Key Lock` 退化为间隙锁；
  - 唯一索引上的范围查询会访问到不满足条件的第一个值为止；
  - 查看锁状态命令：`show engine innodb status\G;`

6.11 `exists` 和 `in` 对比
- `in` 查询时，先查询子查询的表，然后内表和外表做笛卡尔积，在按照查询条件进行筛选；
- `exists` 会先进行主查询，将查询到的数据循环带入子查询校验是否存在；
- 内表大，`exists` 效率高；内表小，`in` 效率高；

<details>
<summary>exists 和 in 语法</summary>

```sql
-- in 查询
select t1.id from tb_data t1 where t1.task_id IN (select t2.id from tb_task t2);

-- exists 查询
select t1.id from tb_data t1 where t1 where EXISTS (select * from tb_task t2 where t1.task_id = t2.id);
```
</details>

### 7. <span id="7">Redis</span>
7.1 常用数据结构
- `String`：分布式锁、存储简单的热点数据；
- `Hash`：用户基本信息，用于抽奖的活动信息(活动的名称，开始时间，结束时间，审核状态等等)；
- `Set`：共同好友，共同喜好，客户的兴趣标签；
- `zSet`：文章点赞排行榜、延迟队列；
- `List`
- `zSet` 底层是 `ziplist` 或者 `skiplist`，`skiplist`(跳表)的优势：
  - 查找单个 key，`skiplist`和红黑树的平均时间复杂度都为`O(logN)`；
  - 跳表的内存占用比红黑树要少，红黑树的每个节点包含 2 个指针（分别指向左右子树），而跳表每个节点包含的指针数目平均为 `1/(1-p)`，Redis 的实现中取 p=1/4，平均每个节点包含 1.33 个指针；
  - 跳表的范围查找比红黑树简单，红黑树找到指定范围的小值之后，还需要通过中序遍历查找其他不超过大值的节点。跳表的范围查找，只需要找到小值之后，对第一层链表进行若干步的遍历即可；
    - `skiplist`和各种平衡树（如 AVL、红黑树等）的元素是有序排列的，而哈希表不是有序的；
    - 所谓范围查找，指的是查找那些大小在指定的两个值之间的所有节点；
  - 跳跃表比红黑树更容易实现，因为红黑树的插入和删除操作，可能需要做一些 rebalance 操作，而跳表的插入和删除只需要修改相邻节点的指针；
- `skiplist(跳表)`是一种有序的数据结构，它通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的；
  - 跳表在链表的基础上，增加了多层级索引，通过索引位置的跳转，实现数据的快速定位；

7.2 Redis 集群
- Redis 集群满足 CAP 中的 `CP`，即一致性和分区容错性；
  - Redis Cluster 集群中，如果某个主节点没有从节点，那么当它发生故障时，集群将完全处于不可用状态。
- 集群类型：
  - Redis Sentinel 着眼于**高可用**；
  - Redis Cluster 着眼于**扩展性**，用于扩展内存，三主三从；
- Redis 有多个节点时，key 存储在哪个节点？
  - 普通 hash 算法：增加或删除节点时，基本上所有的数据都要重建路由；
  - 一致性 Hash 算法 + 虚拟节点：
    - 将 `0 ~ （2^32）- 1`的范围，抽象为一个圆环，使用 CRC16 算法计算出来的哈希值会落在圆环的某个位置；
    - 优势：如果一个节点挂了，只影响此节点到环空间前一个节点（沿着逆时针方向行走遇到的第一个节点）之间的数据，其他不受影响；
    - 问题：在节点太少时，因为节点分布不均匀，造成数据倾斜（被缓存的对象大部分几种缓存在某一台服务器上）；
    - 引入虚拟节点机制，即对每个节点计算多个 hash，每个计算结果位置都放置一个虚拟节点；
  - redis cluster 的 hash slot 算法
    - redis cluster 有固定的 16384(2^14) 个桶，对每个 key 计算 CRC16 的值，然后对 16384 取模，可以获取 key 对应的 hash slot；
- Redis Cluster 故障转移
  - 某节点认为 A 宕机，此时是主观宕机；若集群内超过半数的节点认为 A 挂了，此时 A 就会标记为客观宕机；一旦节点被标记为了客观宕机，集群就会开始执行故障转移；其余正常运行的 master 节点，会进行投票选举，从 A 节点的 slave 节点中选举一个，将其切换成新的 master 对外提供服务。当某个 slave 获得超过半数的 master 节点投票，就成功当选；新的节点会执行 slave no one 来让自己停止复制 A 节点，使自己成为 master，向集群发送 PONG 消息来广播自己的最新状态；
  - Redis Cluster 利用 gossip 协议来实现自身的消息扩散；每个 Redis 节点每秒钟都会向其他的节点发送 PING，然后被 PING 的节点会回一个 PONG；
- “脑裂”：即“大脑分裂”，本来一个“大脑”被拆分成两个或多个。
  - 在 Redis 集群环境中，通常只有一个“大脑”。在网络正常的情况下，可以顺利的选举出Master。但当两个机房之间的网络通信出现故障时，选举机制就有可能在不同的网络分区中选出两个Master。当网络恢复时，这两个Master该如何处理数据同步？又该听谁的？这也就出现了“脑裂”现象。
  - 除了机房分区导致的脑裂现象，网络抖动也可能导致 Master 假死，如果某个 Master 假死，其余的 followers 选举出一个新的 Master。这时，旧的 Master 复活，仍认为自己是 Master，向其他 followers 发出写请求也是会被拒绝的；

![脑裂之前](./images/脑裂之前.jpeg)

![脑裂之后](./images/脑裂之后.jpeg)

- 如何避免“脑裂”
1. 过半原则：在 Master 选举过程中，如果某个节点获得了超过半数的选票，则可以成为 Master；
  - 以上图6台服务器为例来进行说明：half = 6 / 2 = 3，也就是说选举的时候，要成为Leader至少要有4台机器投票才能够选举成功。那么，针对上面2个机房断网的情况，由于机房1和机房2都只有3台服务器，根本无法选举出Leader。这种情况下整个集群将没有Leader。
2. 添加心跳线：
  - 急群众采用多种通信方式，防止一种通信方式失效导致集群中的节点无法通信；

7.3 RDB 和 AOF
- RDB(Redis Database)内存快照，指的是 Redis 内存中的数据在某一刻的状态数据；
  - RDB 采用二进制 + 数据压缩的方式写磁盘，文件体积小，数据恢复速度快；但有可能会丢失数据；
- AOF 日志存储的是 Redis 服务器的顺序指令序列，AOF 只记录对内存进行修改的指令记录；
  - AOF 文件过大，恢复过程会非常缓慢；
- RDB + 增量 AOF 的混合持久化策略；

7.4 Redis 分布式锁
- `set NX key <value>`：占锁成功，若业务代码出现异常或服务器宕机，没有执行删除锁的逻辑，会造成死锁，因此锁需要设置过期时间；
- `set key <value> PX <过期时间> NX`：占锁 + 过期时间，为保证原子性，可以使用 Lua 脚本；过期时间设置为 110ms；
  - `EX`：秒；
  - `PX`：毫秒；

```java
/**
 * 分布式锁
 *
 * 如果超时未解锁，视为加锁线程死亡，其他线程可夺取锁
 */
public boolean setNx(String key, Long lockExpireMils) {
    return (boolean)redisTemplate.execute((RedisCallback)connection -> {
        // 获取锁，并设置过期时间
        return connection.setNX(key.getBytes(), String.valueOf(System.currentTimeMillis() + lockExpireMils + 1).getBytes());
    });
}
```

- 生成随机唯一 id，给锁加上唯一值；
  - 释放锁需要使用 Lua 脚本，包含两步：查询比较锁的值 + 删除锁；

```java
/**
 * 获取不阻塞的锁lua脚本
 */
private static final String NO_BLOCK_LOCK_SCRIPT = "if redis.call('set', KEYS[1], ARGV[1], 'NX', 'EX', ARGV[2]) then return 1 else return 0 end";

/**
 * 释放锁lua脚本
 */
private static final String RELEASE_LOCK_SCRIPT = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
```

- 阻塞获取锁(50ms)
  - 假设 1 万个请求同时去竞争一把锁，可能只有一个请求是成功的，其余 9999 个请求都会失败；
- 抽奖活动中的分布式锁：
  - 个人用户把参与活动的信息，写入到用户表；
  - 加锁是为了提高可靠性，通常 incr 不会出现并发问题，拿到相同的值。但实际集群部署下，临界情况和库存补充时，也可能会出现相同值的情况；
  - 早期在内部测试时，redis 是集群服务，incr 压测十万次，最后的结果总是会差几个，和运维了解到每次调用是网络连接，如果网络请求失败，会自动发起重试。所以可能导致不一定是 10 万次，所以加了 setnx 锁，以保证不超卖；

7.4.1 Redisson
![redisson锁原理](./images/redisson锁原理.jpeg)

- watchdog 机制，看门狗机制来自动延长加锁的时间；

7.5 缓存常见问题
- 缓存雪崩：大量缓存数据在同一时间过期(失效)或 Redis 故障宕机；
  - 均匀设置过期时间；
  - 互斥锁，保证同一时间只有一个请求来构建缓存；
  - 搭建 redis 集群，实现高可用；
- 缓存击穿：热点数据过期，导致缓存失效，大量请求直接访问数据库；
  - 互斥锁；
  - 不给热点数据设置过期时间；
- 缓存穿透：数据既不在缓存中，也不在数据库中；场景：黑客恶意攻击，故意大量访问某些不存在的数据；
  - 非法请求限制；
  - 缓存空值或默认值；
  - 使用布隆过滤器，快速判断数据是否存在，避免通过查询数据库来判断数据是否存在；
- 布隆过滤器：
  - 由初始值为 0 的位图数组和 N 个哈希函数两部分组成；
  - 用 N 个哈希函数分别对数据做哈希计算，得到 N 个哈希值；
  - 将第一步得到的 N 个哈希值，对位图数组的长度取模，得到每个哈希值在位图数组的对应位置；
  - 将每个哈希值在位图数组的对应位置，设置为 1；
  - 查询布隆过滤器，若查询到不存在，则数据库中一定不存在这个数据；若存在，则有可能存在；

7.6 数据库缓存一致性
- 数据库和缓存无法做到强一致，但可以做到的是缓存和数据库达到最终一致，而且不一致的时间窗口能做到尽可能短；
  - 从一致性角度看，采用**更新数据库后删除缓存值**，是更为合适的策略；
  - 读写分离 + 主从复制延迟下，缓存和数据库一致性问题：
    - 缓存延迟双删策略，延迟时间的设置：
    - 延迟时间要大于主从复制的延迟时间；
    - 延迟时间要大于线程 B 读取数据库 + 写入缓存的时间；

7.7 分布式 ID 生成策略
- ID 生成规则的硬性要求：
  - 全局唯一；
  - 趋势递增，MySQL 的 InnoDB 是聚集索引，使用的是 B+ 树结构来存储数据，应尽量使用有序的主键保证数据写入；
  - 含时间戳，可以快速了解分布式 ID 生成的时间，定位问题；
  - 高可用低延迟：ID 生成的速度要快，避免成为系统瓶颈；
- UUID 不能生成顺序、递增的数据，而且长；
- 数据库集群模式：起始值和自增步长，集群多的情况，扩容比较麻烦；
- 雪花算法，生成的是 Long 型 ID，一个 Long 型占 64 bit：
  - 最高位为符号位，不使用；
  - 41 位：用来记录时间戳；
  - 10 位：工作机器 id；
  - 12 位：序列号；
  - 解决时钟回拨问题：美团开源的 Leaf 组件；

![雪花算法结构](./images/雪花算法结构.png)

7.8 Redis 性能高的原因
1. 基于内存：Redis 的大部分操作都在内存中完成；
2. 高效的数据结构：Redis 每种数据类型底层都做了优化，目的是为了追求更快的速度；
3. IO 多路复用：Redis 采用 I/O 多路复用机制处理大量的客户端 Socket 请求；
4. Redis 采用单线程模型，可以避免多线程之间的竞争，省去了多线程切换带来的时间和性能上的开销；

### 8. <span id="8">ElasticSearch</span>

8.1 倒排索引
- 通过分词策略，形成了词与文章的映射关系表，从词出发，记载了这个词在哪些文档中出现过，由两部分组成：词典和倒排表；

![倒排索引_文档](./images/倒排索引_文档.jpeg)

![倒排索引_结果](./images/倒排索引_结果.png)

8.2 ES 创建索引过程

![ES 创建索引](./images/es索引流程.jpeg)

1. 客户端发送请求，往集群某节点写入数据。（如果没有指定路由/协调节点，请求的节点扮演路由节点的角色）；
2. 节点 1 接受到请求后，使用文档`_id`来确定文档属于分片 0，请求会被转到另外的节点，假定分片 0 的主节点位于节点 3 上；
3. 节点 3 在主分片上执行写操作，如果成功，则将请求并行转发到节点 1 和节点 2 的副本分片上，等待结果返回。所有的副本分片都报告成功，节点 3 将想协调节点（节点 1）报告成功，节点 1 向请求客户端报告写入成功；
  - 路由算法：`hash(_routing)%(nums_of_primary_shards)`；

8.3 ES 的检索流程

![ES 检索流程](./images/es检索流程.webp)

1. Client 将请求发送到任意节点 node，此时 node 节点就是协调节点(coordinating node)；
2. 协调节点进行分词等操作后，去查询所有的 shard(primary shard 和 replica shard 选择一个)；
3. 所有 shard 将满足条件的数据 id 排序字段等信息返回给协调节点；
4. 协调节点重新进行排序，截取数据后，获取到真正需要返回数据的 id；
5. 协调节点再次请求对应的 shard（此时有 id，可以直接定位到对应 shard）；
6. 获取到全量数据，返回 Client；

8.4 数据类型
- String 类型
  - `text`：会分词，默认使用 standard 分词器；
  - `keyword`：不会分词；
- 数值类型
- 时间类型
- 复杂类型

8.5 join 联表查询解决方案
- 宽表冗余存储：对每个文档保持一定数量的冗余数据，可以在需要访问时，避免进行关联；
- [mysql 同步到 ES，join 查询解决方案](https://www.infoq.cn/article/1afyz3b6hnhprrg12833)

### 9. <span id="9">消息队列(Kafka)</span>
9.1 MQ 的作用
- 异步、解耦、削峰
- 场景：秒杀场景下，如何应对瞬时高流量并发请求？
  - 引入消息中间件(如 kafka)，梳理业务流程，将耗时的处理环节，由同步处理调整为异步处理；
  - 服务器扩容：增加部署的服务器数量；
  - 增加缓存：提升查询性能；

9.2 Kafka 为什么快？
- Partition 顺序读写，充分利用磁盘特性；
- Producer 生产的数据持久化到 Broker，采用 mmap(Memory Mapped Files) 文件映射，实现顺序的快速写入，mmap 将磁盘文件映射到内存，用户通过修改内存就能修改磁盘文件；
- Consumer 从 Broker 读取数据，采用 `sendfile`，将磁盘文件读到 OS 内核缓冲区后，直接转到 socket buffer 进行网络发送；
- Broker 性能优化：日志记录批处理、批量压缩等；
- Java NIO 对文件映射的支持：
  - 通过调用 `FileChannel.map()` 取得 `MappedByteBuffer`，`MappedByteBuffer` 可以用来实现内存文件映射；
  - 通过 `FileChannel` 的 `transferTo`、`transferFrom` 实现零拷贝；

9.3 Kafka 数据丢失问题
- 生产者数据不丢失：
  - `ack=0`：producer 不等待 broker 同步完成的确认，继续发送下一条消息；
  - `ack=1`(默认)：producer 要等到 leader 成功收到数据并得到确认后，才发送下一条消息；
  - `ack=-1`：等待所有 follower 收到消息并得到确认，才发送下一条数据；
- 消费者数据不丢失：取消自动提交，变为手动提交；
- broker 消息不丢失：每个 broker 中的 partition 设置副本数至少为 3 个；
- 新增消息发送表：
  - 当生产者发送消息之前，会往该表中写入一条数据，status 标记为待确认；如果消费者读去消息后，消费成功，则调用生产者的 API 更新 status 为已确认；
  - 新增定时 job，每隔一段时间检查一次消息发送表，若 5 min 后还有状态是待确认的消息，则认为该消息已经丢失了，需要重新发送消息；

![消息发送表](./images/消息发送表.png)

9.4 消息的顺序问题
- Kafka 同一个 partition 中可以保证顺序，但不同的 partition 无法保证顺序；

9.5 重复消息(程序的幂等设计)
- 数据库增加防重字段，即唯一索引，在插入重复数据时，会抛出 `DuplicateKeyException`；先查询 Redis 缓存，若存在，说明已经处理过了，直接丢掉；若不存在，再执行插入数据库的操作；

9.6 数据一致性
- 为了性能考虑，使用 MQ + 重试表来保证数据的最终一致性；
- 若消费者处理失败，则写入到重试表，定时任务进行重试，并限制重试次数，最大为 5 次；

### 10. <span id="10">Dubbo</span>
- SPI(Service Provider Interface)是一种服务发现机制，本质是将接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类。这样可以在运行时，动态为接口替换实现类；

10.1 Java 原生 SPI
- 缺点：
  - 使用一次，就需要一次性实例化所有实现类，没有缓存功能；
  - 获取指定实现类，需要遍历所有实现类，逐个比对；

10.2 Dubbo 的 SPI
- Dubbo 采用“微内核 + 插件”的方式，实现了**对扩展开放，对修改关闭**；
- Dubbo 几乎所有的功能组件都是基于 **拓展机制(SPI)** 实现的，其有点：
  - 通过O(1)的时间复杂度来获取指定的实例对象；
  - 增加缓存功能，每次获取的为同一个对象；
  - 基于 Dubbo 的 SPI 加载机制，让整个框架的接口与具体实现完全解耦；

### 11. <span id="11">设计模式</span>
11.1 单例模式

<details>
<summary>单例模式</summary>

```java
// 懒汉式：使用时，才加载
public class Singleton {
  private static Singleton instance;

  private Singleton(){}

  public static synchronized Singleton getInstance() {
    if (null != instance) {
      return instance;
    }
    instance = new Singleton();
    return instance;
  }
}

// 饿汉式：提前加载
public class Singleton {
  private static Singleton instance = new Singleton();
  
  private Singleton(){}

  public static Singleton getInstance() {
    return instance;
  }
}

// 双重锁检查机制
public class Singleton {
  private static volatile Singleton instance;

  private Singleton(){}

  public static Singleton getInstance() {
    if (null != instance) {
      return instance;
    }
    synchronized(Singleton.class) {
      if (null == instance) {
        instance = new Singleton(); // 这里可能会存在重排序
      }
    }
    return instance;
  }
}

// 创建一个对象，可以分解为如下的三行伪代码
memory = allocate();  // 1，分配对象的内存空间
ctorInstance(memory); // 2，初始化对象
instance = memory;    // 3，设置 instance 指向刚分配的内存地址

// 2 和 3 可能发生重排序，有可能会访问到未被初始化的对象，抛出 NullPointerException
```
</details>

![重排序](./images/重排序.webp)

- 策略模式
  - 用于解耦策略的定义、创建、使用；
  - 多种营销策略：直减、满减、折扣、N 元购；

<details>
<summary>策略模式</summary>

```java
// 策略的定义
public interface DiscountStrategy {
  double calDiscount(Order order);
}

// 具体的策略实现类：普通订单、团购订单、促销订单
// 省略 NormalDiscountStrategy、GrouponDiscountStrategy、PromotionDiscountStrategy

// 策略的创建
public class DiscountStrategyFactory {
  private static final Map<OrderType, DiscountStrategy> strategies = new HashMap<>();

  static {
    strategies.put(OrderType.NORMAL, new NormalDiscountStrategy());
    strategies.put(OrderType.GROUPON, new NormalDiscountStrategy());
    strategies.put(OrderType.PROMOTION, new NormalDiscountStrategy());
  }

  public static DiscountStrategy getDiscountStrategy(OrderType type) {
    return strategies.get(type);
  }
}

// 策略的使用
public class OrderService {
  public double discount(Order order) {
    OrderType type = order.getType();
    DiscountStrategy discountStrategy = DiscountStrategyFactory.getDiscountStrategy();
    return discountStrategy.calDiscount(order);
  }
}
```
</details>

- 责任链模式
  - 多个处理器(Handler)依次处理同一个请求；

<details>
<summary>责任链模式</summary>

```java
// 定义抽象父类
public abstract class Handler {
  protected Handler successor = null;

  public void setSuccessor(Handler successor) {
    this.successor = successor;
  }

  public final void handle() {
    boolean handled = doHandle();
    if (successor != null && !handled) {
      successor.handle();
    }
  }

  protected abstract boolean doHandle();
}

// 具体的处理器类
public class HandlerA extends Handler {
  @Override
  protected boolean doHandle() {
    boolean handled = false;
    // ... 具体业务逻辑
    return handled;
  }
}

public class HandlerB extends Handler {
  @Override
  protected boolean doHandle() {
    boolean handled = false;
    // ... 具体业务逻辑
    return handled;
  }
}

// HandlerChain
public class HandlerChain {
  private Handler head = null;
  private Handler tail = null;

  public void addHandler(Handler handler) {
    handler.setSuccessor(null);

    if (head == null) {
      head = handler;
      tail = handler;
      return;
    }

    tail.setSuccessor(handler);
    tail = handler;
  }

  public void handle() {
    if (head != null) {
      head.handle();
    }
  }
}

// 使用举例
public class Application {
  public static void main(String[] args) {
    HandlerChain chain = new HandlerChain();
    chain.addHandler(new HandlerA());
    chain.addHandler(new HandlerB());
    chain.handle();
  }
}
```
</details>

### 12. <span id="12">算法</span>
- 二分查找时间复杂度：`O(logN)`；
- 快排的平均时间复杂度和最优时间复杂度：`O(N * logN)`，最差时间复杂度：`O(N^2)`；
- 求数组 a[] 和 b[] 的交集，先遍历 a[]，存 HashMap，然后判断是否存在；
- 用栈实现队列：使用 2 个栈
  - 栈：先进后出，队列：FIFO；

<details>
<summary>用栈实现队列</summary>

```java
    Stack<Integer> input = new Stack<>();
    Stack<Integer> output = new Stack<>();

    public MyQueue() {}

    public void push(int x) {
        input.push(x);
    }

    public int pop() {
        peek();
        return output.pop();
    }

    public int peek() {
        if (output.empty()) {
            while (!input.empty()) {
                output.push(input.pop());
            }
        }
        return output.peek();
    }

    public boolean empty() {
        return input.empty() && output.empty();
    }
```
</details>

- 数组中第 K 大的数字

<details>
<summary>数组中第 K 大的数字</summary>

```java
  public static int findKthLargest(int[] nums, int k) {
        return quickSelect(nums, 0, nums.length - 1, k);
    }

  private static int quickSelect(int[] nums, int low, int high, int k) {
      int pivot = low;

      // use quick sort's idea
      // put nums that are <= pivot to the left
      // put nums that are > pivot to the right
      for (int j = low; j < high; j++) {
          if (nums[j] <= nums[high]) {
              swap(nums, pivot++, j);
          }
      }
      swap(nums, pivot, high);

      // count the nums that are > pivot from high
      int count = high - pivot + 1;
      // pivot is the one
      if (count == k) {
          return nums[pivot];
      }
      if (count > k) {
          return quickSelect(nums, pivot + 1, high, k);
      }
      return quickSelect(nums, low, pivot - 1, k - count);
  }

  private static void swap(int[] nums, int a, int b) {
      int temp = nums[a];
      nums[a] = nums[b];
      nums[b] = temp;
  }
```
</details>

- leetcode 第一题
  - 给定一个整数数组 nums 和一个目标值 target，请在该数组中找出和为目标值的两个整数，并返回他们的数组下标

<details>
<summary>代码</summary>

```java
public static int[] twoSum(int[] nums, int target) {
  Map<Integer, Integer> map = new HashMap<>();

  for (int i = 0; i < nums.length; i++) {
    int partnerNum = target - nums[i];
    if (map.containsKey(parterNum)) {
      return new int[]{map.get(partnerNum), i};
    }
    map.put(nums[i], i);
  }
  return null;
}
```
</details>

- 100 G文件出现次数最多的 100 个 IP
  - 先 `%1000`，将 IP 分到 1000 个小文件中；
  - 对每个小文件中的 ip 进行 HashMap 计数统计；
  - 归并或最小堆，依次处理每个小文件中的 Top 100,得到最后结果；

### 13. <span id="13">领域驱动设计(DDD)</span>
#### 13.1 贫血模型和充血模型
- MVC 架构，Repository 层负责数据访问，Service 层负责业务逻辑，Controller 层负责暴露接口；
- 贫血模型(Anemic Domain Model)：
  - 将数据和业务逻辑分离，如 Service 层的数据和业务逻辑，被分割为 BO 和 Service 两个类；
    - 弊端：数据和业务逻辑分离之后，数据本身的操作就不受限制了，任何代码都可以随意修改数据；
- 充血模型(Rich Domain Model)：
  - 数据和业务逻辑被封装同一个类中；
  - 基于充血模型的 DDD 开发模式中，Service 层包含 Service 类和 Domain 类两部分；Domain 就相当于贫血模型中的 BO。不过，Domain 中既包含数据，也包含业务逻辑，而 Service 类变得非常单薄；
  - Service 层的作用：
    - Service 类负责与 Repository 交互；将流程性的代码逻辑（比如从 DB 中取数据、映射数据）与领域模型的业务逻辑解耦，让领域模型更加可复用；
    - Service 类负责跨领域模型的业务聚合功能；
    - Service 类负责一些非功能性的工作，比如幂等、事务、发邮件、发消息、记录日志、调用其他系统的 RPC 接口等；
  - 基于充血模型的 DDD 开发模式中，Controller 层和 Repository 层仍采用贫血模型，
    - 因为 Repository 的 Entity 传递到 Service 层之后，就会转化成 BO 或者 Domain 来继续后面的业务逻辑。Entity 的生命周期到此就结束了，所以并不会被到处任意修改；
    - 而 Controller 层实体一种 DTO(Data Transfer Object，数据传输对象)，它不包含业务逻辑、只包含数据。所以，将 VO 设计成贫血模型也是比较合理的。
- 总的来说，基于贫血模型的传统开发模式，重 Service 轻 BO；基于充血模型的 DDD 开发模式，轻 Service 重 Domain；

#### 13.2  贫血模型和充血模型的开发流程
- 基于贫血模型的传统开发模式：
  - 大部分都是 SQL 驱动的开发模式。
  - 我们接到一个后端接口的开发需求的时候，就去看接口需要的数据对应到数据库中，需要哪张表或者哪几张表，然后思考如何编写 SQL 语句来获取数据。之后就是定义 Entity、BO、VO，然后模板式地往对应的 Repository、Service、Controller 类中添加代码；
  - SQL 都是针对特定的业务功能编写的，复用性差；当我开发另一个业务功能的时候，只能重新写个满足新需求的 SQL 语句，这就可能导致各种长得差不多、区别很小的 SQL 语句满天飞；
  
- 基于充血模型的 DDD 开发模式的开发流程，在应对复杂业务系统的开发的时候更加有优势；
  - 首先理清楚所有的业务，定义领域模型所包含的属性和方法。领域模型相当于可复用的业务中间层。新功能需求的开发，都基于之前定义好的这些领域模型来完成；
  - 越复杂的系统，对代码的复用性、易维护性要求就越高，我们就应该花更多的时间和精力在前期设计上。而基于充血模型的 DDD 开发模式，正好需要我们前期做大量的业务调研、领域模型设计，所以它更加适合这种复杂系统的开发；

#### 13.3 不同架构之间的区别
- DDD 分层架构(抽奖系统所使用的)
  - 用户接口层：为前端提供接口；
  - 应用层：
    - 协调多个聚合的服务和领域对象完成服务编排和组合；
  - 领域层：核心业务逻辑，包含聚合根、实体、值对象、领域服务；
    - 当领域中的某些功能，单一实体（或者值对象）不能实现时，领域服务可以组合聚合内的多个实体（或者值对象），实现复杂的业务逻辑；
  - 基础层：为其他各层提供通用的技术和基础服务，包括第三方工具、驱动、消息中间件、网关、缓存以及数据库等；
- 分层架构根据耦合的紧密程度分为：严格分层架构和松散分层架构：
  - 严格分层架构：任何层只能对于其直接下方的层产生依赖；服务是逐层对外封装或组合的，依赖关系清晰；
  - 松散分层架构：允许某层与其任意下方的层发生依赖；服务的依赖关系比较复杂且难管理，甚至容易使核心业务逻辑外泄；

![DDD 分层架构](./ddd/images/DDD分层架构.jpg)

- 六边形架构（又称“端口和适配器模式”）
  - 核心理念是：**应用通过端口与外部进行交互**；
  - 六边形架构将系统分为内六边形和外六边形：
  - 内六边形是领域业务层，负责业务逻辑的组织和实现；
  - 外六边形是适配器层，负责与外部进行交互；
    - 主动适配器：主动调用应用核心端口，触发应用执行某项活动；
    - 被动适配器：被应用核心端口调用来执行动作，实现应用定义的端口；
      - 如端口RespositoryInterface -> 被动适配器Mysql实现MysqlRepositoryImpl -> 调用Mysql数据库

![六边形架构](./ddd/images/六边形架构.png)

- 洋葱架构和整洁架构中的层就像洋葱片一样，体现了分层设计的思想，以及高内聚、低耦合的设计特性；

- 洋葱架构(Onion Architecture)
  - 洋葱架构在六边形架构的基础上，将内层(业务逻辑层)进一步划分：
    - 洋葱架构的最外层包括：用户接口、基础设施、测试策略；
    - 洋葱架构的内层，即原来六边形架构中的业务逻辑层：
      - 应用服务层
      - 领域服务层
      - 领域模型层
    - 在洋葱架构里面，同心圆代表应用软件的不同部分，从里到外依次是领域模型，领域服务，应用服务和外层的基础设施和用户终端；
  - 洋葱架构明确了依赖方向：
    - 外层依赖内层；
    - 内层不知道外层的存在；
    - 且在不破坏依赖方向的前提下，外层亦可以直接调用任一内层（不一定是相邻接的2层）；

![洋葱架构](./ddd/images/洋葱架构.png)

- 整洁架构(Clean Architecture)
  - 与洋葱架构相比，整洁架构调整如下：
    - `Application Services` 调整为 `Use Cases`；
    - `Domain Services` 和 `Domain Model` 调整为 `Entities`；

![整洁架构](./ddd/images/整洁架构.png)

- CQRS
  - 读写分离

### 14. <span id="14">系统设计相关</span>
14.1 Redis 实现延迟队列
- 延迟队列使用场景：
  - 保证抽奖活动准时开启和关闭，提前 5 分钟把活动扫描到缓存中，然后使用延迟队列来处理；
  - 中奖的优惠券或立减卡快过期时，提醒用户尽快消费；
- 基于 Redis 的延迟队列

![超时任务获取](./images/超时任务获取.png)

![redis存储设计](./images/redis存储设计.png)

a. 将定时任务存储到任务库，定时扫描任务库，即扫描开始时间小于 5min 的任务；
b. 为了支持大量消息的堆积，需要把消息分散存储到很多槽中。在消息存储时，采用对指定数据或者消息体哈希进行与运算得到槽位置；
c. StoreQueue 结构采用 zSet，SlotKey 设计为`#{topic}_#{index}`，Redis 的 zSet 中的数据按照分数排序，实现定时消息的关键就在于如何利用分数、如何添加消息到 zSet、如何从 zSet 中弹出消息。定时消息将时间戳作为分数，消费时每次弹出分数大于当前时间戳的一个消息；
d. PrepareQueue：为了保障每条消息至少消费一次，消费者不是直接 pop 有序集合中的元素，而是将元素从 StoreQueue 移动到 PrepareQueue 并返回消息给消费者，等消费成功后再从 PrepareQueue 中删除，或者消费失败后从 PrepareQueue 重新移动到 StoreQueue，以二阶段消费的方式进行处理；
  - PrepareQueue 的 SlotKey 设计为 `prepare_#{topic}_#{index}`，以**秒级时间戳*1000 + 重试次数**作为分数；

14.2 秒杀系统设计
- 特点：瞬时高并发，一般在秒杀时间点前几分钟，用户并发量才真正突增；
- 方案：
  - 页面静态化，活动页面大多数内容是固定的，如活动名称、描述、活动图片，减少不必要的服务端请求；
  - CDN(Content Delivery Network)，即内容分发网络，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率；
  - 活动开始之前，秒杀按钮置灰，不可点击。只有到了秒杀时间点，秒杀按钮才会自动点亮，变成可点击的；
  - 秒杀活动，属于大量用户抢少量商品，只有极少部分用户能够抢成功，典型**读多写少**场景，从 Redis 缓存查询库存；
    - 缓存问题：缓存雪崩、缓存击穿、缓存穿透；
    - 使用 Redis 分布式锁，进行库存扣减；
  - 梳理业务流程，使用 MQ，异步处理次要流程；
    - MQ 消息丢失：消息发送表 + MQ 任务补偿；
    - MQ 消息重复消费：redis 缓存 + 唯一索引；
  - 对非法请求限流：
    - 限流方式：
      - 基于 Nginx 限流
      - 基于 Redis 限流
    - 限流内容：
      - 对同一用户限流；
      - 对同一 IP 限流；
      - 对接口限流；
      - 增加验证码；
      - 提高业务门槛，如只有等级到达 3 级以上的用户，才有资格参与秒杀活动；
    - 限流算法：
      - 令牌桶：本质是速率控制；令牌桶的保护主要是丢弃请求，即使系统还能处理，只要超过指定的速率就丢弃，除非此时动态提高速率；
      - 漏桶：本质是总量控制；漏桶的保护是尽量缓存请求，缓存不下去才丢弃请求；

![漏桶算法](./images/限流_漏桶算法.jpeg)

![令牌桶算法](./images/限流_令牌桶算法.jpeg)

14.3 抽奖助手
- 以抽奖为整个路线看，需要有 3 个步骤，参与、执行、兑现：
  - 也就是对应的活动、抽奖、奖品；
- 领域建模过程：
  - 通过事件风暴来划分限界上下文和领域，以及上下文之间关系；
  - 进一步分析每个上下文内部，识别出哪些是实体，哪些是值对象：
    - 实体：拥有唯一标识，重要的不是其属性，而是其延续性和标识，如：客户基础信息、活动信息；
    - 值对象：通过对象属性值来识别对象，它将多个相关属性组合为一个概念整体，如地址、抽奖规则；
  - 对实体、值对象进行关联和聚合，划分出聚合的范围和聚合根；
  - 为聚合根设计存储，并思考实体、值对象的创建方式；
  - 在工程中实践领域模型，并在实践中检验模型的合理性，倒推模型不足的地方并重构；
  - 抽奖系统的领域划分，最初来自 PRD 流程的拆解设计，划分出界限关系和所属问题域空间；拆分出：规则引擎、活动管理、抽奖策略、奖品发放等领域服务；
  - 活动管理聚合根：
    - 活动信息的查询，活动的上线、下线；
    - 活动的抽奖策略、奖品配置管理；
    - 活动库存的扣减、活动奖品的数量管理(返回被抽完的奖品)；
- 领域细分为不同的子域，子域根据其自身重要性和功能属性，划分为：核心域、支撑域、通用域：
  - 核心域：活动管理(发布部署、活动状态流转、参与活动)，抽奖策略，奖品发放；
  - 通用域：规则引擎；
  - 支撑域：分布式 ID 生成；

- 防腐层的职责：一个上下文通过一些适配和转换与另一个上下文交互。
  - 适配性：两个或更多不同的子系统（或限界上下文）具有不同的领域模型，需要对外部上下文的方法进行一次转义。例如同一个字段，在外部系统叫做“地瓜”，在我们系统叫做“红薯”，我们需要通过防腐层来转化成我们系统约定的字段名；
  - 封装性：我们可能会在防腐层中对外部多个接口进行封装处理，从而返回我们系统中真正需要的参数，而外部多个接口的封装处理将沉淀到防腐层中。
  - 旧版单体应用迁移到新版微服务系统，但是迁移计划发生在多个阶段，新旧系统之间的集成需要维护。
    - 若要与旧系统进行交互操作，新应用程序可能需要支持过时的基础结构、协议、数据模型、API、或其他不会引入新版应用程序的功能。
  - 防御性：防腐层往往属于下游限界上下文，用以隔绝上游限界上下文可能发生的变化；

![防腐层](./images/防腐层1.png)

![防腐层2](./images/防腐层2.png)

14.3.1 参数配置
- QPS：每秒能够响应的查询次数；
- TPS：每秒处理的事务数，一个事务是指一个客户端向服务器发送请求然后服务器做出响应的过程；
- RT：平均响应时间；
- 并发数：系统能同时处理的请求/事务数量；
  - QPS = 并发数/RT;
- 关于流量评估的场景(28法则)
  - 假如系统有 300 万用户，那么每天来点击页面的占比 20%，也就是 60 万用户访问；
  - 假设平均每个用户点击 50 次，那么总共有 3000 万的PV；
  - 一天 24 个小时，平均活跃时间段算在 5 个小时内(24 * 20%)，那么 5 个小时预计有 2400 万点击(3000 * 80%)，也就是平均每秒 1333 个请求；
  - 1333 是一个均值，按照电商类峰值的话，一般是 3~4 倍均值量，也就是 5 个小时每秒 5000 个请求(QPS = 5000)；
  - TP99 为 110 ms，TP999 为 140ms，平均响应时间在 120 毫秒左右。RPC 抖动和数据库查询并发较高时，TP999 max 会达到 400 毫秒，对于超时 250 毫秒的 RPC 会采用重试一次，降低 TP999 最大响应时长，增加系统接口可用率；
  - 机器配置为 4 核 8G，接口响应时间大约为 200\~300ms，每秒可以扛住 300\~500 个请求；
  - 并发数 = QPS/RT，即 5000QPS * 120/1000 = 600 并发，600/150=4 台机器；
  - 重点日期开展活动，会有 3.5 倍左右的扩容数量服务器，即 12 台虚拟机；
  - 抽奖服务是一套服务，打包成 jar 镜像直接部署；
- 数据库表：
  - 活动信息表、抽奖策略配置表、奖品信息表(每个活动的奖品)、用户抽奖单记录表(记录用户的抽奖历史)；
- 分布式事务：MQ + 任务调度补偿，保证事务的最终一致；
  - 分布式事务性能较差；
  - 记录用户领取活动的表和活动配置表不在一个库，不能做一个事务，所以用户领取活动完毕后，发送 MQ 去更新数据库中的库存；

- 活动处理流程

![活动处理流程](./images/抽奖流程.png)

- 抽奖奖品：
  - 虚拟奖品：优惠券、积分、兑换码、购物卡；
  - 实物奖品：华为手表、手机、抽纸、消毒酒精等；
- 标签：
  - 性别、年龄、首单、忠诚度以及点击、搜索、喜好；
  - 是否订阅指定公众号；
  - 拉新用户存款(存款金额 + 至少存一个月)；
  - 是否使用指定银行进行云缴费(水电燃气)；
- 在活动过程中，发生 Redis 故障，数据库和缓存不一致问题，则增加挡板，活动紧急下线，并提醒用户“抽奖活动太火爆，运营正在紧急补货中，稍后上架”；

14.3.2 ShardingSphere
- 分库分表：解决连接数瓶颈，以及单表数据量过大问题；
  - 根据 userId 进行分库分表；
  - 为什么不按时间进行分库分表？
    - 抽奖系统为实时业务数据，如果基于时间片处理带来的问题是，当一个用户请求进来做业务，很难定位这个用户所属的库表，也包括一些基于人维度的事务处理；
  - 根据业务体量的发展，设定一个 2 ~ 3年增量的分库分表规模，分为 4 个库，16 张表/库；
  - 把分库分表的数据同步到 ES，方便对 B 端提供数据汇总查询；

![shardingsphere执行流程](./images/shardingsphere执行流程.png)

- 版本：4.1.1
- 配置内容：
  - 数据源(datasource.type、driver-class-name、username、password)
  - 主键策略：snowflake
  - 真实库分布
  - 真实表分布
  - 默认分库策略、默认分表策略
  - 自定义分库分表策略：整数模除法散列

```java
// 库表索引
int dbIdx = idx / dbRouterConfig.getTbCount() + 1;
int tbIdx = idx - dbRouterConfig.getTbCount() * (dbIdx - 1);
```

- 严格雪崩标准
  - 在密码学中，雪崩效应是密码算法的理想属性，通常是分组密码和密码散列函数，其中如果输入发生轻微变化(例如，翻转单个位)，输出会发生显著变化(例如，50%输出位翻转)；
- 简单来说，当我们对数据库从 8 库 32 张表扩容到 16 库 32 张表的时候，每一个表中的数据总量都应该以 50% 的数量进行减少，这才是合理的。
- 测试步骤：
  - 准备 10 万个单词作为样本数据；
  - 对比测试除法散列、乘法散列、斐波那契散列；
  - 基于条件 1、2，对数据通过不同的散列算法分两次路由到 8 库 32 表和 16 库 32 表中，验证每个区间内数据的变化量，是否在 50% 左右；
  - 准备一个 excel 表，来做数据的统计计算。


14.3.3 规则引擎 Drools
  - 基本功能：将传入的数据或事实与规则的条件进行匹配，并确定是否以及如何执行规则；
  - 优点：可以通过决策表动态生成规则脚本，把开发人员从大量的规则代码的开发维护中释放出来，把规则的维护和生成交由业务人员；
  - 抽奖系统自研了一套决策表的解决方案，将 DRL 规则脚本中的条件和规则抽象成结构化数据进行存储，先通过可视化页面录入规则，然后由系统自动生成规则脚本；

14.4 线上事故
- 事故现象：线上监控突然报警，CPU 占用高，拖垮整个服务，用户看到可以参与抽奖，但一点击抽奖，页面异常。增加挡板，活动紧急下线；
- 事故原因：抽奖助手的分布式锁，最开始是基于一个活动号 ID 进行锁定，秒杀时锁定这个 ID，用户抽完奖后进行释放；所有的用户都抢这一个 key，其中有一个用户在业务逻辑处理过程中发生异常，释放锁失败；
- 优化：独占竞态锁调整为分段静态锁，将**活动 ID + 库存编号**作为动态锁标识，所有进来的用户抢的是自增 key，锁的颗粒度小，某个用户发生异常，不影响其他人；
