[官方文档](http://dubbo.apache.org/zh-cn/docs/user/quick-start.html)
[他人博客](https://www.jianshu.com/p/c23c82a8fcfc)
####        
1. 调用链
2. 服务注册与发现流程
3. 服务注册
4. 服务发现
5. 降级
6. 集群容错
7. 负载均衡
8. 传输协议
9. 序列化协议
10. 通信框架
11. 监控中心

##### ExtensionLoader
##### [事件机制](https://www.cnblogs.com/java-zhao/p/8436460.html)
##### Dispatcher-线程派发
接收消息的线程被称为IO线程，消息类型分为请求、响应、连接事件、断开事件等

* all：所有类型的消息都派发到线程池
* direct：所有类型的消息都**不**派发到线程池，全部在 IO 线程上直接执行
* message：只有请求和响应消息派发到线程池，其它消息均在 IO 线程上执行
* execution：只有请求消息派发到线程池，不含响应。其它消息均在 IO 线程上执行
* connection：在 IO 线程上，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池

默认配置下，Dubbo 使用 all 派发策略

##### 插件
1. Qos 服务质量
##### 计数器限流 [TPSLimiterFilter](https://www.jianshu.com/p/7112a8d3d869)
#### 其他常见面试题
1. 为什么使用dubbo
