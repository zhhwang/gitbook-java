# redis

## keys

- 二进制安全， 可以存储图片信息
- 不要太长，不要太短
- 推荐的模式：冒号，点，中横线
    - user:1000
    - comment:1234:reply-to
    - comment:1234:reply.to
- 每个key的value最大为512M

## 数据类型

### strings
mapping a string to another string

- set
- get
- incr
- mset

#### 适用

- 原子计数器
- append string
- 随机访问容器
    - getrange
    - setrange

### list

- lpush
- lpop
- rpush
- rpop
- lrange
- rrange
- blpush
- blpop

### hash

- hadd
- hget
- hgetall
- hmget

### set

- sadd
- smembers
- sismember

### sorted set

impl by skip list and hash table

- zadd 
- zrange
- zrevrange

### bitmaps

- getbit
- setbit

### hyperLogLog

- pfadd
- pfcount

### streams

##### Redis

- redis为什么快
- 适用场景
- 与memcached的区别
- 与mongo的区别
- 几种集群模式
- 动态扩容
