
### tornado Web框架
* [generator和coroutine的介绍](https://juejin.im/post/5c13245ee51d455fa5451f33)
* [装饰器](https://www.jianshu.com/p/ee82b941772a)
* 
*
*
*
进程：有自己的内存空间
线程：多个线程共享进程的内存空间

无复杂计算的web应用的cpu使用率不高，是io密集成场景

python协程原理/缺点
* 无须线程上下文切换
* 无须原子操作及同步的开销
* 方便切换控制流，简化编程 模型
1. 无法利用多核CPU优势（python脚本语言，运行效率低，CPU密集任务更适合c语言开发）
2. 阻塞会阻塞掉整个应用程序


python是动态语言，脚本语言

python GIL, 为什么有GIL 还需要threading
python gevent协程调度原理/缺点


gc
- 引用计数器
 - 非容器对象
- 标记清除
    - 容器对象
    - root链表
    - unreachable链表
- 分代收集
    - 幼年：幼年代中 分配对象个数 - 释放对象个数 > threshold0 触发垃圾收集
    - 青年：青年代中 分配对象个数 - 释放对象个数 > threshold1 触发垃圾收集
    - 老年: 老年代中 分配对象个数 - 释放对象个数 > threshold2 触发垃圾收集



脚本语言
编译型语言

动态语言
静态语言
