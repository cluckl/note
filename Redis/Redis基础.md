## 简介

Redis是一个基于内存实现的支持Key-Value等多种数据结构的存储系统。

应用场景：

* 会话缓存
* 消息队列
* 订阅发布

### 基本使用

1. 选择数据库: `select index`
2. 获取当前数据库的所有key：`keys *`
3. 删除当前数据库的全部内容：`flushdb`
4. 删除所有数据库的全部内容：`flushall`
5. 判断当前key是否存在：`exists key`
6. 移除当前key：`move key`
7. 设置key的过期时间：`expire key seconds  `
8. 显示key的过期时间：`ttl [key]` 
9. 查看key的类型：`type key`

### Redis速度快的原因

1. 绝大部分请求是纯粹的**<u>内存操作</u>**，非常快速
2. 采用**<u>单线程</u>**，避免了不必要的上下文切换和竞争条件导致的加锁解锁操作
3. 使用 **<u>I/O 多路复用模型</u>**
4. 数据结构和数据操作简单，Redis 中的数据结构是专门进行设计的

## 数据类型

- **String（字符串）**
- **List（列表）**
- **Hash（字典）**
- **Set（集合）**
- **Sorted Set（有序集合）**
- Geospatial（地理位置）
- Hyperloglog（基数统计）
- Bitmap（位图）

### String

String是简单的 key-value 键值对，value 不仅可以是 String，也可以是数值。

底层实现：String在redis内部存储默认是一个字符串，被redisObject所引用，当遇到`incr`、`decr`等操作时会转成**<u>数值型</u>**进行计算。

### List

list对应的value的底层是一个**<u>双向链表</u>**，可以从头部或尾部添加或者删除元素。

List可以用于实现一个轻量级消息队列。

### Hash

hash：一个键值对的集合，是一个string类型的field和value的映射表，适合用于存储对象，如field存储用户ID，value存储序列化的用户信息。

底层实现：在hash的成员数量比较少时为了节省内存会采用类似**<u>一维数组</u>**的方式存储数据，当成员数量增大时会自动转成HashMap。

### Set

set是字符串类型的无序集合，可进行集合运算。

底层实现：set 的内部实现是一个value为空的HashMap，通过计算hash的方式快速排重。

### zset

zset(sorted set)：string类型的有序集合，每个元素需要指定一个score，根据score对元素进行升序排序（如果多个元素有相同的分数，则以字典序进行升序排序），适用于**<u>排名</u>**等场合。

底层实现：使用HashMap和**<u>跳跃表(SkipList)</u>**来保证数据的存储和有序，HashMap存放成员到score的映射，而跳跃表存放所有的成员，排序依据是HashMap里存的score,使用跳跃表的结构可以获得比较高的查找效率，并且在实现上比较简单。