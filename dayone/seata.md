
### 事务协调器TC
### 事务管理器TM
### 资源管理器RM

### AT模式
#### 前提
- 基于支持本地 ACID 事务的关系型数据库
- Java 应用，通过 JDBC 访问数据库。
#### 流程
- GlobalTransaction注解，TM向TC注册全局事务
- TM开始执行事务，将事务上下文向RM传递
- RM启用了数据库代理
    - 执行sql时写入undo日志
    - commit proxy：获取到全局锁，获取到本地锁，向TC注册分支事务，执行db commit，修改分支事务状态为commit(此时全局事务未提交，
        db已提交，db修改已经可被其他事务所见，所以全局事务是读未提交的隔离级别)
    - rollback proxy：执行db rollback，向上抛出异常
- TM：
    - 执行未发现异常: 通知TC commit
    - 执行发现异常：通知TC rollback
- TC:
    - commit: 通知所有注册的分支删除undo日志
    - rollback: 通知所有注册的分支执行undo日志，然后删除undo日志
    
### 隔离级别
ps: 没有看到commit proxy中是怎么获取本地锁和全局锁的

- 数据库是读已提交
- 全局事务是读未提交

### MT模式
    
