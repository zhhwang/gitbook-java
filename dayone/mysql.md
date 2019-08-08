##### mysql
1. 怎样做性能优化
2. 索引类型
3. 索引为什么快
4. 聚簇索引和非聚簇索引，联合索引
5. 悲观锁，乐观锁，间隙锁,
6. 主从复制原理，数据丢失问题，同步延时问题

### 事务
### 锁
#### [Innodb中的锁](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
##### 行级锁-共享锁和排他锁
* 共享锁(S)：允许事务中使用此类型的锁读取一行数据，两个事务可以同时获取一个行的共享锁
* 排他锁(X)：允许事务中使用此类型的锁更新或删除一行数据，两个事务不可以同时获取一个行的排他锁
##### 表级锁-意向锁
* 共享意向锁(IS)：获取一个表中的某一行的共享锁之前，必须先获取这个表的共享意向锁
* 排他意向锁(IX)：获取一个表中的某一行的排他锁之前，必须先获取这个表的排他意向锁
![共享锁和排他锁的兼容性](../assets/compatibilitySX.png)
##### 记录锁
##### 间隙锁
##### Next-key锁
##### 插入意向锁
##### 自增锁
##### Predicate Locks for Spatial Indexes
### 索引
### 集群
### SQL
### 分库分表中间件
#### myCat
#### [sharding-jdbc](https://shardingsphere.apache.org/document/current/cn/overview/)
### 数据库连接池
#### druid
#### hikari
### 持久层框架
#### jpa
#### mybatis
