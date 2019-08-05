#### 什么是线程安全
1. 原子性
2. 可见性
3. 有序性

#### 实现线程安全的几种方式
1. 不可变对象
2. 同步互斥
3. 非阻塞同步
4. 栈封闭和ThreadLocal

### 线程

#### 线程状态
new
runnable
waiting
time_waiting
blocked
terminated

#### start()和run()

#### yield()

#### join()

#### 线程中断
1. 中断标志位
2. 响应中断
3. 不响应中断

#### 线程间通信
1. volatile和synchronized关键字
2. 等待/通知机制
3. 管道输入/输出流
4. Thread.join()

### volatile

#### 特性
1. 可见性
2. 有序性
3. 为什么不能保证原子性

### synchronized

#### 实现原理

#### 对synchronized的优化

#### synchronized的使用
1. 修饰(this)
2. 修饰(A.class)
3. 修饰普通方法
4. 修饰静态方法

### synchronized与lock的区别

### Lock

#### 重入锁

#### 重入读写锁

### AQS原理

### JUC

#### 原子类

#### 阻塞队列

##### ArrayBlockList

##### LinkedBlockList

#### 并发容器

#### 线程池

#### 并发工具类

##### CountDownLatch

##### CyclicBarrier

##### Semaphore

#### 并发框架

### Java内存模型

#### [happens-before规则](https://www.jianshu.com/p/d52fea0d6ba5)
Java Memory Model and Thread Specification中定义了如下happens-before规则：
* 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
* 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
* volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
* 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
* start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
* join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。

* 程序中断规则：对线程interrupted()方法的调用先行于被中断线程的代码检测到中断时间的发生。
* 对象finalize规则：一个对象的初始化完成（构造函数执行结束）先行于发生它的finalize()方法的开始。

### 其他常见面试题
1. 交替打印奇数和偶数
2. 线上问题定位
