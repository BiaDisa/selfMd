## 复习经历

因为之前就深入学习过，所以总的复习时间也不长，大概是一周左右，后面是通过边面试边查漏补缺的方式来补短板。

> 前两天的复习内容：

### Java 基础

- 面向对象特性：封装，多态（动态绑定，向上转型），继承
- 泛型，类型擦除
- 反射，原理，优缺点
- `static`，`final` 关键字
- `String`，`StringBuffer`，`StringBuilder`底层区别
- BIO、NIO、AIO
- `Object` 类的方法
- 自动拆箱和自动装箱

### Java 集合框架

- List ：`ArrayList`、`LinkedList`、`Vector`、`CopyOnWriteArrayList`
- Set：`HashSet`、`TreeSet`、`LinkedHashSet`
- Queue：`PriorityQueue`
- Map：`HashMap`，`TreeMap`，`LinkedHashMap`
- fast-fail，fast-safe 机制
- 源码分析（底层数据结构，插入、扩容过程）、线程安全。

### Java 虚拟机

- 类加载机制、双亲委派模式、3 种类加载器（`BootStrapClassLoader`，`ExtensionClassLoader`，`ApplicationClassLoader`）
- 运行时内存分区（PC，Java 虚拟机栈，本地方法栈，堆，方法区（永久代，元空间））
- JMM：Java 内存模型
- 引用计数、可达性分析
- 垃圾回收算法：标记-清除，标记-整理，复制
- 垃圾回收器：比较，区别（Serial，ParNew，Parallel Scavenge ，CMS，G1）Stop The World
- 强、软、弱、虚引用
- 内存溢出、内存泄漏排查
- JVM 调优，常用命令

### Java 并发

- 三种线程初始化方法（`Thread`、`Callable`，`Runnable`）区别

- 线程池（`ThreadPoolExecutor`，7 大参数，原理，四种拒绝策略，四个变型：Fixed，Single，Cached，Scheduled）

- - `Synchronized`：

  - 使用：方法（静态，一般方法），代码块（this，`ClassName.class`）

  - 1.6 优化：锁粗化，锁消除，自适应自旋锁，偏向锁，轻量级锁

  - 锁升级的过程和细节：无锁->偏向锁->轻量级锁->重量级锁（不可逆）

  - 重量级锁的原理（`monitor`对象，`monitorenter`,`monitorexit`）

  - `ReentrantLock`：和`Synchronized`区别？（公平锁、非公平锁、可中断锁....）、原理、用法

  - 有界、无界任务队列，手写`BlockingQueue`。

  - 乐观锁：CAS（优缺点，ABA 问题，DCAS）

  - 悲观锁：

  - `ThreadLocal` ：底层数据结构：`ThreadLocalMap`、原理、应用场景。

  - `Atomic` 类（原理，应用场景）

  - AQS：原理、`Semaphore`、`CountDownLatch`、`CyclicBarrier`

  - `Volatile`：原理：有序性，可见性

    > 第三天的复习内容：

### MySQL

- 架构：Server 层，引擎层（缓存，连接器，分析器，优化器，处理器）
- 引擎：InnoDB，MyISAM，Memory 区别
- 聚簇索引，非聚簇索引区别（从二叉平衡搜索树复习（AVL，红黑树）到 B 树，最后 B+树）
- MySQL、SQL 优化方法
- 覆盖索引，最左前缀匹配
- 当前读，快照读
- MVCC 原理（事务 ID，隐藏字段，Undo，ReadView）
- Gap Lock、Next-Key Lock、Record Lock
- 三大范式

### SQL

- 常用 SQL
- 连接：自连接，内连接（等值，非等值，自然连接），外连接（左，右，全）
- Group BY 和 Having
- Explain

> 第四天的复习内容：

### Spring

- AOP 原理（JDK 动态代理，CGLIB 动态代理）和 IOC 原理
- Spring Bean 生命周期
- SpringMVC 原理
- SpringBoot 常用注解
  - @Transactional
  - @Mapper
  - @RequestMapping,@RequestBody
  - @Async
    - https://blog.csdn.net/clementad/article/details/47403185
  - 

### 设计模式

- 三种类型：创建、结构、行为
- 单例模式：饿汉，懒汉，DCL
- 简单工厂，工厂方法，抽象工厂
- 代理模式
- 装饰器模式
- 观察者模式
- 策略模式
- 迭代器模式
- ....

> 第五天的复习内容：

### 计算机网络

- OSI 模型、TCP/IP 模型

- TCP 和 UDP 区别

- TCP 可靠性传输原理：重传、流量控制、拥塞控制、序列号与确认应达号、校验和

- 三次握手、四次挥手过程、原理

- timewait、closewait

- HTTP

- - 报文格式
  - 1.0 1.1 2.0
  - 状态码
  - 无状态解决（Cookie Session 原理）

- HTTPS

- - CA 证书
  - 对称加密
  - 非对称加密

- DNS 解析过程，原理

- IP 协议、ICMP 协议（Ping、Tracert）、ARP 协议、路由协议

- 攻击手段与防范：XSS、CSRF、SQL 注入、DOS、DDOS

> 第六天的复习内容：

### 操作系统：

- 进程、线程和协程区别
- 进程通信方式（管道，消息队列，共享内存，信号，信号量，socket）
- 进程调度算法（先来先服务，短作业优先，时间片轮换，多级反馈队列，优先级调度）
- 内存管理：分页（页面置换算法：手写 LRU）、分段、虚拟内存

> 第七天和以后的复习内容：

每天做点刷算法题(剑指 offer、LeetCode 面试 Hot 题) +查漏补缺。