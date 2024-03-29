---
title: Core Java
toc: true
date: 2022-06-09 14:15:36
tags: core-java
categories: java
---

# Collections

## HashMap

- 整体设计?是否线程安全?为什么?
  - 数组+链表+红黑树,hash 冲突时用链表,>8 树化,<6 链表化
  - 线程不安全,未加锁
- hash 怎么计算的?为什么这么做?
  - 高 16bit 不变，低 16bit 和高 16bit 做了一个异或
  - 数组索引计算 i = (table.length - 1) & hash
- 怎么扩容的?为什么使用 2 次幂扩展?
  - bucket 扩充为 2 倍，之后重新计算 index.因为我们使用的是 2 次幂的扩展(指长度扩为原来 2 倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动 2 次幂的位置
- 并发问题?死循环
  - 在 HashMap 并发进行 Resize 的过程中会出现环形链表，导致 get() 操作死循环。

## ConcurrentHashMap

- 整体设计?是否线程安全?为什么?跟 HashTable 的差异
  - table 数组＋单向链表＋红黑树,读不加锁,写加锁,锁粒度细到数组元素级
  - HashTable 只是简单加 synchronized,性能较差
- 怎么保证性能
  - 读不加锁,用 volatile 保证可见性
  - 写加锁,用 table 数组元素作为锁,实现了对每一行数据进行加锁，进一步减少并发冲突的概率

## BlockingQueue

- 功能特性?
  - 在任意时刻，当有界 BlockingQueue 队列元素放满之后，所有的元素都将在放入的时候阻塞。无界 BlockingQueue 没有任何容量限制，容量大小始终是 Integer.MAX_VALUE
  - 主要用于 生产者-消费者 的队列
- 整体设计?
  - ArrayBlockingQueue: 底层由数组存储的有界队列。遵循 FIFO，所以在队首的元素是在队列中等待时间最长的，而在队尾的则是最短时间的元素。新元素被插入到队尾，队列的取出 操作队首元素

# Concurrent

## Thread

- 线程（Thread）是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务
- 线程状态:
  - 新建（New）: 新建的Thread，尚未开始。
  - 运行（Runable）: 包含操作系统线程状态中的Running、Ready，也就是处于正在执行或正在等待CPU分配时间的状态。
  - 无限期等待（Waiting）: 处于这种状态的线程不会被分配CPU时间，等待其他线程唤醒。
  - 限期等待（Timed Waiting）: 处于这种状态的线程不会被分配CPU时间，在一定时间后会由系统自动唤醒。
  - 阻塞（Blocked）: 在等待获得排他锁。
  - 结束（Terminated）: 已终止的线程。
- java 线程池的核心参数，以及运行原理，如何确定线程池线程数大小，为什么
- ThreadPoolExecutor 构造函数重要参数：
  - corePoolSize : 核心线程数线程数定义了最小可以同时运行的线程数量。
  - maximumPoolSize : 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
  - workQueue: 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。
  - keepAliveTime：当线程池中的线程数量大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime 才会被回收销毁；
  - unit ：keepAliveTime 参数的时间单位。
  - threadFactory ：executor 创建新线程的时候会用到。
  - handler ：饱和策略。

## volatile

- 作用是什么?怎么实现的?
  - 1)保证可见性;2)禁止重排序
    - 当程序执行到 volatile 变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见，在其后面的操作肯定还没有进行；
    - 在进行指令优化时，不能将在对 volatile 变量访问的语句放在其后面执行，也不能把 volatile 变量后面的语句放到其前面执行。
  - 计算机内存模型:cpu(寄存器)->三级缓存->主存
  - 为了解决缓存不一致性问题，通常来说有以下两种解决方法：1)通过在总线加 LOCK#锁的方式(效率低,早期采用);2)通过缓存一致性协议(MESI 协议)当 CPU 写数据时，如果发现操作的变量是共享变量，即在其他 CPU 中也存在该变量的副本，会发出信号通知其他 CPU 将该变量的缓存行置为无效状态，因此当其他 CPU 需要读取这个变量时，发现自己缓存中缓存该变量的缓存行是无效的，那么它就会从内存重新读取。
  - 无法保证原子性.在 Java 1.5 的 java.util.concurrent.atomic 包下提供了一些原子操作类，即对基本数据类型的 自增（加 1 操作），自减（减 1 操作）、以及加法操作（加一个数），减法操作（减一个数）进行了封装，保证这些操作是原子性操作。atomic 是利用 CAS 来实现原子性操作的（Compare And Swap），CAS 实际上是利用处理器提供的 CMPXCHG 指令实现的，而处理器执行 CMPXCHG 指令是一个原子性操作。
  - 实现机制:内存屏障
    - 观察加入 volatile 关键字和没有加入 volatile 关键字时所生成的汇编代码发现，加入 volatile 关键字时，会多出一个 lock 前缀指令
    - lock 前缀指令实际上相当于一个 内存屏障（也称内存栅栏），内存屏障会提供 3 个功能：
      - 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
      - 它会强制将对缓存的修改操作立即写入主存；
      - 如果是写操作，它会导致其他 CPU 中对应的缓存行无效。

## synchronized

### 1.6前为重量级锁

### 1.6后进行优化

- 偏向锁
  - 优点: 加锁、解锁无额外消耗，和非同步方式近似
  - 缺点: 如果竞争线程多，会有额外锁撤销的消耗
  - 适用场景: 基本没有线程竞争的场景
- 轻量级锁
  - 优点: 竞争线程不会阻塞，使用自旋等待
  - 缺点: 如果长时间不能获取锁，会消耗CPU
  - 适用场景: 少量线程竞争，且线程持有锁时间不长
- 重量级锁
  - 优点: 竞争线程被阻塞，减少CPU空转
  - 缺点: 线程阻塞，响应时间长
  - 适用场景: 很多线程竞争，锁持有时间长

- jvm优化
  - 锁消除: 经过逃逸分析，去除不可能存在共享资源竞争的锁
  - 锁粗化:  JVM 将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁，避免频繁的加锁解锁操作。

## Lock

### ReentrantLock 作用是什么?怎么实现的?

#### Synchronized vs ReentrantLock

- synchronized 是 Java 关键字，ReentrantLock 基于 AQS 的 API 层面的互斥锁
- ReentrantLock可以设置等待超时时间
- ReentrantLock可进行公平锁与非公平锁设置
- ReentrantLock可绑定多个 Condition
- synchronized 不需要手动释放锁
- synchronized 可以修饰方法、代码块

- 悲观锁，乐观锁，优缺点，CAS 有什么缺陷，该如何解决

## Java 内存模型(JMM)

- 注意，为了获得较好的执行性能，Java 内存模型并没有限制执行引擎使用处理器的寄存器或者高速缓存来提升指令执行速度，也没有限制编译器对指令进行重排序。也就是说，在 java 内存模型中，也会存在缓存一致性问题和指令重排序的问题。
- Java 内存模型规定所有的变量都是存在主存当中（类似于前面说的物理内存），每个线程都有自己的工作内存（类似于前面的高速缓存）。线程对变量的所有操作都必须在工作内存中进行，而不能直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。
- 原子性:即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。
  - 只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作
  - 在 32 位平台下，对 64 位数据的读取和赋值是需要通过两个操作来完成的，不能保证其原子性。但是好像在最新的 JDK 中，JVM 已经保证对 64 位数据的读取和赋值也是原子性操作了。
  - 如果要实现更大范围操作的原子性，可以通过 synchronized 和 Lock 来实现。由于 synchronized 和 Lock 能够保证任一时刻只有一个线程执行该代码块，那么自然就不存在原子性问题了，从而保证了原子性。
- 可见性:可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。
  - 当一个共享变量被 volatile 修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值
  - synchronized 和 Lock 能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。
- 有序性:即程序执行的顺序按照代码的先后顺序执行。
  - 指令重排序，一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。
  - 处理器在进行重排序时是会考虑指令之间的数据依赖性，如果一个指令 Instruction 2 必须用到 Instruction 1 的结果，那么处理器会保证 Instruction 1 会在 Instruction 2 之前执行。
- happens-before 原则（先行发生原则）
  - 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
  - 锁定规则：一个 unLock 操作先行发生于后面对同一个锁额 lock 操作
  - volatile 变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
  - 传递规则：如果操作 A 先行发生于操作 B，而操作 B 又先行发生于操作 C，则可以得出操作 A 先行发生于操作 C
  - 线程启动规则：Thread 对象的 start()方法先行发生于此线程的每个一个动作
  - 线程中断规则：对线程 interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
  - 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过 Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
  - 对象终结规则：一个对象的初始化完成先行发生于他的 finalize()方法的开始

## ThreadLocal

## AQS

# JVM

## 运行时数据区

### 程序计数器

- 子节码的行号指示器
- 线程私有
- 未规定 OutOfMemoryError 情况

### 栈(虚拟机栈/本地方法栈)

- 线程私有
- 虚拟机栈(执行子节码)
  - 栈帧: 存储局变量表、操作数栈、动态连接、方法出口等信息
  - 超出栈深度,抛出 StackOverflowError
  - 若栈可以动态扩展,可能抛出 OutOfMemoryError
- 本地方法栈
  - 执行本地方法,hot-spot 将其合并到虚拟机栈

### 方法区

- 所有线程共享
- 存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据
- 永久代迁移为元空间(MetaSpace)
- 无法分配内存时抛出 OutOfMemoryError
- 运行时常量池
  - 可动态添加常量
  - 无法分配内存时抛出 OutOfMemoryError

### 堆

- 所有线程共享
- 无法扩展堆时抛出 OutOfMemoryError

### 直接内存(堆外内存)

- 不属于虚拟机运行时数据区,通过 Native 函数直接分配
- 不受堆大小限制,但超出物理机内存限制时会抛出 OutOfMemoryError
- GC 通过 Cleaner#clean 间接管理

## Garbage Collect

### Young GC

- Serial
  - 单线程,标记-复制算法
- ParNew
  - 一款多线程的收集器，采用标记-复制算法，主要工作在 Young 区，可以通过 -XX:ParallelGCThreads 参数来控制收集的线程数，整个过程都是 STW 的，常与 CMS 组合使用。
- Parallel Scavenge
  - 基于标记-复制
  - 关注吞吐量
    - -XX MaxGCPauseMillis 参数允许的值是一个大于 0 的毫秒数   收集器将尽力保证内存回收花费的时间不超过用户指定值。
    - -XX GCTimeRatio 参数的值则应当是一个大于 0 小于 100 的整数   也就是垃圾收集时间占总时间的比率,相当于吞吐量的倒数。
    - -XX +UseAdaptiveSizePolicy,vm 自适应调节策略

### Old GC

- CMS(Concurrent Mark and Sweep)
  - 以获取最短回收停顿时间为目标，采用“标记-清除”算法，分 4 大步进行垃圾收集，其中初始标记和重新标记会 STW ，多数应用于互联网站或者 B/S 系统的服务器端上，JDK9 被标记弃用，JDK14 被删除，详情可见 JEP 363。
  - 收集过程:
    - 初始标记(CMS initial mark)_stw_
    - 并发标记(CMS concurrent mark)
    - 重新标记(CMS remark)_stw_
    - 并发清除(CMS concurrent sweep)
- Serial Old
  - 单线程,标记-整理算法
- Parallel Old
  - 多线程,标记-整理算法

### 分区收集

- G1(Garbage First)
- ZGC
- Shenandoah

## JVM 调优

- 逃逸分析技术,标量替换
- 常用的 JVM 调优参数

### 常用工具

- 命令行终端
  - 标准终端类：jps、jinfo、jstat、jstack、jmap
  - 功能整合类：jcmd、vjtools、arthas、greys
- 可视化界面
  - 简易：JConsole、JVisualvm、HA、GCHisto、GCViewer
  - 进阶：MAT、JProfiler

### GC 问题评价标准

- 延迟（Latency）：也可以理解为最大停顿时间，即垃圾收集过程中一次 STW 的最长时间，越短越好，一定程度上可以接受频次的增大，GC 技术的主要发展方向。
- 吞吐量（Throughput）：应用系统的生命周期内，由于 GC 线程会占用 Mutator 当前可用的 CPU 时钟周期，吞吐量即为 Mutator 有效花费的时间占系统总运行时间的百分比，例如系统运行了 100 min，GC 耗时 1 min，则系统吞吐量为 99%，吞吐量优先的收集器可以接受较长的停顿。
- 目标: 一次停顿的时间不超过应用服务的 TP9999，GC 的吞吐量不小于 99.99%

## 类加载

### 类的生命周期

- 加载 Loading:查找并装载类型的二进制数据。
- 验证 Verification:确保被导入类型的正确性
- 准备 Preparation(连接 Linking):为类变量分配内存，并将其初始化为默认值。
- 解析 Resolution(连接 Linking):把类型中的符号引用转换为直接引用。
- 初始化 Initialization(连接 Linking):把类变量初始化为正确的初始值。
- 使用 Using
- 卸载 Unloading

### 类加载过程

- 加载 Loading:查找并装载类型的二进制数据。
- 连接:执行验证、准备以及解析(可选)
  - 验证 Verification:确保被导入类型的正确性
  - 准备 Preparation:为类变量分配内存，并将其初始化为默认值。
  - 解析 Resolution(连接 Linking):把类型中的符号引用转换为直接引用。
- 初始化 Initialization(连接 Linking):把类变量初始化为正确的初始值。
  - Clinit 方法
    - 对于静态变量和静态初始化语句来说：执行的顺序和它们在类或接口中出现的顺序有关。
    - 并非所有的类都需要在它们的class文件中拥有<clinit>()方法， 如果类没有声明任何类变量，也没有静态初始化语句，那么它就不会有<clinit>()方法。
    - 只有那些需要执行Java代码来赋值的类才会有<clinit>()
    - jvm保证clinit的线程安全性,一个类型只会被初始化一次。

### 类加载器

- Bootstrap ClassLoader：jvm实现的一部分(C++),负责加载核心 Java 库.
- java.lang.ClassLoader
  - Extention Classloader: 加载java扩展库
  - Application Classloader：根据 Java应用程序的类路径（ java.class.path 或 CLASSPATH 环境变量）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader() 来获取它。
  - 自定义类加载器：可以通过继承 java.lang.ClassLoader 类的方式实现自己的类加载器，以满足一些特殊的需求而不需要完全了解 Java 虚拟机的类加载的细节。

### 双亲委派机制

- 全盘负责：指当一个 ClassLoader 装载一个类的时，除非显式地使用另一个 ClassLoader ，该类所依赖及引用的类也由这个 ClassLoader 载入。
- 双亲委托机制：指先委托父装载器寻找目标类，只有在找不到的情况下才从自己的类路径中查找并装载目标类。
- 优点: 保证类的层级性与唯一性
- 破坏双亲委派模型:
  - 重写loadClass或findClass
  - Thread Context Loader加载SPI服务代码
  - OSGi热部署




## 参考资料
> - []()
> - []()
