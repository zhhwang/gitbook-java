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
##### 索引锁-记录锁
* 记录锁是在索引上加锁，阻止其他事务添加、修改、删除，如下图，只有当前事务可以操作c1=10的行，将不允许这个表出现新的c1 = 10的行
````$xslt
SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE

````
* 如果一个表没有索引，InnoDB会为这个记录锁创建一个隐藏的聚簇索引
##### 索引锁-间隙锁
* 间隙锁是在两个索引记录的间隙范围加锁，或者对before一个索引记录加锁，或者对after一个索引记录加锁。如下图，将不允许其他事务在表t中插入c1的值为15的行
 ````$xslt
SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE
````
* 一个间隙可以为一个索引，多个索引，或者空
* 间隙锁是性能和并发相互平衡的产物，用于部分事务隔离级别
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
