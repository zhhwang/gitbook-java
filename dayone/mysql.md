##### mysql
1. 怎样做性能优化
2. 索引类型
3. 索引为什么快
4. 聚簇索引和非聚簇索引，联合索引
5. 悲观锁，乐观锁，间隙锁,
6. 主从复制原理，数据丢失问题，同步延时问题

#### [Innodb中的锁](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
##### 行级锁-共享锁和排他锁
* 共享锁(S)：允许事务中使用此类型的锁读取一行数据，两个事务可以同时获取一个行的共享锁
* 排他锁(X)：允许事务中使用此类型的锁更新或删除一行数据，两个事务不可以同时获取一个行的排他锁
##### 表级锁-意向锁
* 共享意向锁(IS)：获取一个表中的某一行的共享锁之前，必须先获取这个表的共享意向锁
* 排他意向锁(IX)：获取一个表中的某一行的排他锁之前，必须先获取这个表的排他意向锁
![共享锁和排他锁的兼容性](../assets/compatibilitySX.png)
##### 索引锁-记录锁
* 记录锁是在索引上加锁
````sql
SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE;
# 阻止其他事务添加, 修改, 删除, 只有当前事务可以操作c1=10的行, 将不允许这个表出现新的c1 = 10的行

````
* 如果一个表没有索引，InnoDB会为这个记录锁创建一个隐藏的聚簇索引
##### 索引锁-间隙锁
* 间隙锁是在两个索引记录的间隙范围加锁，或者对before一个索引记录加锁，或者对after一个索引记录加锁。如下图，将不允许其他事务在表t中插入c1的值为15的行
 ````sql
SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE
````
* 一个间隙可以为一个索引，多个索引，或者空
* 间隙锁是性能和并发相互平衡的产物，用于部分事务隔离级别
##### Next-key锁
- 记录锁 + 间隙锁
##### 插入意向锁
##### 自增锁
##### Predicate Locks for Spatial Indexes

#### 不同sql语句使用的锁
##### select
在隔离级别 可串行化&&autocommit=0时会自动转为select...for share
##### select for share/update
select for share/update 也被叫做locking read

- select for share
    是一把共享锁，适合读数据，阻塞后来的排他锁
    - 可以多个事务同时获取
    - 共享锁的升级
        如果只有一个事务获取了共享锁，并在事务中修改了数据，共享锁会升级为排他锁
    - 死锁
        如果有多个事务获取了共享锁，其中一个事务想要修改数据，需要升级为排他锁，将会等他所有其他事务释放掉共享锁;
        如果其中有两个事务想要修改数据，都需要升级为排他锁，会互相等待对方释放排他锁，造成死锁
    - ！！！不建议对select for share获取了排他锁的数据进行修改
        
- select for update
    是一把排他锁，适合修改数据，阻塞后来的共享锁和排他锁
    - 每个时刻只有一个事务可以修改
    - 不阻塞普通的select语句，普通的select语句不需要获取任何一把锁

##### insert
    - 排他锁、record lock
    - 多个事务同时insert同一个unique key，出现了duplicate-key error，可能会造成死锁
##### update
    排他锁
##### delete
    排他锁
##### 索引对锁的影响
对于locking read、update、delete
- 查询条件是唯一索引
    只使用record lock
- 查询条件是其他索引
    使用record lock或next-key lock

### 事务
#### ACID
- Atomicity:
    
    主要涉及InnoDB的事务
    - autocommit
    - commit
    - rollback
- Consistency
    
    主要涉及InnoDB在崩溃时处理和保护数据
    - 双写缓存
    - 崩溃恢复
- Isolation
    主要涉及InnoDB事务和隔离级别
- Durability
    主要涉及与机器硬件配置的交互
    - innodb_doublewrite 配置是否打开双写缓存
    - innodb_flush_log_at_trx_commit 
        - 0 每秒执行写日志并flush到磁盘
        - 1 事务提交时写日志并flush到磁盘
        - 2 事务提交时写日志，每秒flush到磁盘
    - sync_binlog: 
        - 0 不开启binary日志
        - 1 每次事务提交前写binary日志
        - N N个binary日志提交group之后写binary日志
    - innodb_file_per_table： 每个表一个文件
    - 等等
#### 隔离级别
隔离级别升高，锁粒度增加，并发数量减少.
隔离级别从低到高分别为：
- 读未提交
    - 脏读：普通SELECT 语句可能会读到旧的版本的数据, 读取了其他事务未提交的数据
    - locking reads、update、delete与读已提交相同
    
- 读已提交
    - 不可重复读：普通SELECT每次都读取最新的其他事务已经提交的数据
    - locking reads、update、delete使用record lock, 会出现幻读
    - 外键约束和多重key检查使用gap lock
    - 仅支持行级别的binary日志
    - update与delete
        - 如果没有索引, 只锁需要修改的行
        - 如果有索引, 锁index record
    
- 可重复读
    - 普通SELECT在第一次读数据建立snapshot快照，再次读取时只读快照, 
    - 读取不到其他事务添加、修改、删除的数据, 当前事务的修改、删除会影响到当前事务看不到的数据
    - locking reads、update、delete
        - 查询条件是unique索引，只使用record lock
        - 其他查询条件，使用gap锁或next-key锁
    - update与delete
        - 锁住所有扫描的行
    
- 可串行化
    - 普通SELECT在autocommit=0时会转化为select ... for share
    - locking reads、update、delete与可重复读相同

##### 隔离级别对锁哪些row的影响
对于locking read、update、delete
- 读已提交或读未提交
    只锁定最终定位的row
- 可重复读或可串行化
    锁定所有扫描的row
    
#### 死锁

##### 一个死锁的例子：

```sql
# 客户端A插入一条数据, 开启事务, 获取了row i=1的共享锁
mysql> CREATE TABLE t (i INT) ENGINE = InnoDB;
Query OK, 0 rows affected (1.07 sec)

mysql> INSERT INTO t (i) VALUES(1);
Query OK, 1 row affected (0.09 sec)

mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM t WHERE i = 1 FOR SHARE;
+------+
| i    |
+------+
|    1 |
+------+

# 如果此时客户端B想要删除row i=1
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> DELETE FROM t WHERE i = 1;
# 客户端B需要拿到row i=1的排他锁才能删除数据, 由于row i=1已被客户端A获取了共享锁, 客户端B进入获取row i=1锁的等待队列, 等待客户端A释放共享锁


# 而此时客户端A也想要删除row i=1
mysql> DELETE FROM t WHERE i = 1;
ERROR 1213 (40001): Deadlock found when trying to get lock;
try restarting transaction
# 于是出现了死锁, 是因为客户端想要删除, 必须要把对row i=1的共享锁升级为排他锁, 然而客户端B已经在这之前申请了排他锁, 客户端A的共享锁无法升级, 形成了循环等待

```

##### 死锁探测和回滚
- innodb_deadlock_detect=ON 
    如果死锁探测开启，发现死锁会不断回滚最小修改的事务，直到死锁解开（可能回滚一个也可能回滚多个事务）
 
- innodb_lock_wait_timeout
    如果死锁探测未开启， 超时时间之后获取不到锁，将会回滚
    
##### 如果减少死锁的发生
- 使用SHOW ENGINE INNODB STATUS查看最近一个死锁记录和原因
- 使用innodb_print_all_deadlocks查看所有死锁
- 随时准备再次开启由于死锁而回滚的事务
- 尽快提交事务以减少碰撞，不要保留一个未提交事务的session太长时间
- 如果在使用读锁（select ... for share/update), 使用读已提交隔离级别
- 如果在事务中修改多个表或一个表的多个行，要使这些操作的顺序保持一致
- 使用合适的索引，将会减少被lock的行数
- 尽量少使用锁，使用读已提交的隔离级别，每次都可以读取最新提交的数据
- 最后手段是使用表锁

#### mysql对duplicate-key的处理

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

# questions
* 数据库连接池的连接数量与cpu核数相同？
