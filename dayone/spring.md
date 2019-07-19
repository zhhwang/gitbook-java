### [spring](https://www.cnblogs.com/binarylei/p/10198698.html)

### Spring自定义类加载器
    OverridingClassLoader及其子类
### Spring事件
#### Spring事件机制
#### Spring内置事件
##### ContextRefreshedEvent
    ConfigurableApplicationContext的refresh()方法，是在所有bean都已经被加载，Bean后处理都已被探测和激活，单例模式的Bean已经预实例化，
    ApplicationContext已经可用的情况下。只要context没有被closed，refresh可以被多次触发，比如XmlWebApplicationContext支持的热刷新
##### ContextStartedEvent
    ConfigurableApplicationContext的refresh的start方法，用于stop之后的重启
##### ContextStoppedEvent
    ConfigurableApplicationContext的refresh的stop方法
##### ContextClosedEvent
    ConfigurableApplicationContext的refresh的close方法，之后不可以refresh和restart
##### RequestHandledEvent
    web应用中通知所有bean一次http请求已经结束
#### 自定义事件

### MVC
### IOC
    1. bean作用域
        1. singleton：单例
        2. prototype：每次创建新对象
        3. request
        4. session
        5. global session
### AOP
    1. jdk动态代理
    2. cglib动态代理
### 事务
### spring boot
