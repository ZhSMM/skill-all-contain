# Redis命令

[Redis命令参考](http://redisdoc.com/hash/hset.html)

数据类型：

- string：字符串
- hash：哈希
- list：列表
- set：集合
- zset（sorted set）：有序集合

常用命令key管理：

- `keys *`：返回满足的所有键，可以模糊查询，如`keys a*`
- exists key：是否存在指定key，存在返回1，不存在返回0
- expire key second：设置key过期时间 ，单位为秒 s
- del key：删除某个key
- ttl key：查看key剩余时间，key不存在返回-2，存在但没有设置生存时间返回-1，否则返回以s为单位的剩余生存时间
- persist key：取消过去时间
- PEXPIRE key milliseconds：修改key的过期时间为毫秒
- select：选择数据库 默认16个，0-15，设计成多个数据库是为了安全与备份
- move key dbindex：将当前数据库的key转移到其他数据库
- randomkey：随机返回一个key
- rename key key2：重命名
- echo ：打印
- dbsize：查看数据库的key数量
- info：查看数据库信息
- config get *：实时存储收到的请求，返回相关的配置
- flushdb：清空当前数据库
- flushall：清空所有数据库
- type key：返回key所存储的值的类型

Expire key second应用场景：

- 限时的优惠活动
- 网站的数据缓存（定时更新的数据，如排行榜）
- 手机验证码
- 限制网站访客访问频率

key的命名建议：

redis单个key允许存入512M大小

- key不要太长，尽量不要超过1024字节，不仅消耗内存，也会降低查找效率

- key也不要太短，会降低可读性

- 在一个项目中，key最好使用统一的命名模式，如

  ```
  user:id:name
  user:01:zhangs
  用：分隔
  ```

- key名称区分大小写

### String类型

- String类型是Redis最基本的数据类型，一个键最大能存储512MB
- String类型是最简单的k-v类型，value不仅可以是string，也可以是数字，是包含很多种类型的特殊类型
- string类型是二进制安全，意思是redis的string可以存储任何数据，比如序列化的对象进行存储，如图片进行二进制存储

#### 命令

- set key value：多次赋值会覆盖
- SETNX key value ：如果key不存在，则设值并返回1，如果key存在，则不设值返回0；（解决分布式锁 方案之一）
- setex key  10  value：设置key过期时间和值
- setrange string range value：替换字符串

#### 取值语法

- get key：取值，不存在返回nil
- getrange key start end：获取存储在key中字符串的子字符串，[start ，end]
- getbit key offset：获取key指定偏移量的位
- getset：getset key value：设置新值并返回旧值，key不存在返回nil
- strlen key：返回key存储字符串长度

#### 删值语法

- del key：删除指定的key

#### 批量

- 批量写：mset k1 v1 k2 v2 ...一次性写入多个
- 批量读：mget k1 k2 ...一次性读取多个

自增：incr key：key中存储值加一，key不存在会被初始化为0然后再执行incr操作；

自增：incrby key +增值：自增指定值

自减：decr key /decrby key +减值

字符串拼接：append key value：为key追加value值，不存在则赋值

#### 应用场景

1. string通常用于保存单个字符串或JSON字符串数据
2. 因string二进制安全，可以保存图片文件
3. 计数器（常规计数：微博数、粉丝数）

### Hash类型

#### 简介

Redis hash是一个string类型的field和value的映射表。一个key可对应多个field，一个field对应一个value。将一个对象存储为hash类型，较于每个字段都存储成string类型更能节省内存。新建一个hash对象时开始是用zipmap(又称为small hash)来存储的。这个zipmap其实并不是hash table，但是zipmap相比正常的hash实现可以节省不少hash本身需要的一些元数据存储开销。尽管zipmap的添加，删除，查找都是O(n)，但是由于一般对象的field数量都不太多。所以使用zipmap也是很快的,也就是说添加删除平均还是O(1)。如果field或者value的大小超出一定限制后，Redis会在内部自动将zipmap替换成正常的hash实现.。这个限制在redis.conf中配置如下：

```
hash-max-zipmap-entries 512
hash-max-zipmap-value 64
```

#### 操作

##### 赋值

- hset key field value：为指定key设置field/value
- hmset key field value [field1 value1]：为指定key同时设置多个field/value

##### 取值

- hget key field：获取key 中 field字段的值
- hmget key field [field1] ：获取key中多个field的值
- hgetall key：获得key所有field/value

- hkeys key ：获取所有哈希表中的field
- hlen key：获取key中的field数

##### 删除

- hdel key field [field1...]：删除key中一到多个field

##### 其他

- hsetnx key field value：只有字段field不存在时才设值
- hincrby key field increment：设置key的field字段增加increment
- hincrbyfloat key field increment：为key中field字段中的某个浮点增加increment
- hexists key field：查看哈希表key中，field是否存在

##### 应用场景

- 存储一个对象

- 为什么不用string存储？

  1）json格式：如user：1   {"name":"zhans","age":18}

  - 增加了序列化和反序列化开销
  - 且再修改其中某项数据时，需要取回整个对象，且需要进行并发保护，引入CAS等复杂问题

  2）直接存对象成员，以用户ID+对应属性作为唯一标志存取，会浪费大量空间。

### 常用的Redis客户端

在spring boot 2之后，对redis的支持，默认采用lettuce。

- jedis
- Redisson
- lettuce：基于Netty框架，方法调用异步