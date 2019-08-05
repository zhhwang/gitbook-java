#### 什么是线程安全
* 原子性
* 可见性
* 有序性

#### 实现线程安全的几种方式
* 不可变对象
* 同步互斥
* 非阻塞同步
* 栈封闭和ThreadLocal

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
* 中断标志位
* 响应中断
* 不响应中断

#### 线程间通信
* volatile和synchronized关键字
* 等待/通知机制
* 管道输入/输出流
* Thread.join()

### volatile

#### 特性
* 可见性
* 有序性
* 为什么不能保证原子性

### synchronized

#### 实现原理

#### 对synchronized的优化

#### synchronized的使用
* 修饰(this)
* 修饰(A.class)
* 修饰普通方法
* 修饰静态方法

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
* 交替打印奇数和偶数
* 线上问题定位
