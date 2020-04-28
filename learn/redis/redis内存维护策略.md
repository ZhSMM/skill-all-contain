# redis内存维护策略

redis作为优秀的中间缓存件，时常会存储大量数据，即使采用了集群部署来动态扩容，也应该即时整理内存，维护系统性能。

在redis中有两种解决方案：

1. 为数据设置超时时间

   ```
   设置过期时间
   keys * 查看所有key
   expire key time(单位：s)  ---- 最常使用的方式
   setex(String key ,int seconds,String value)   ----字符串独有的方式
   ```

   - 除了字符串有自己独有的设置过期的方法外，其他方法均需依靠expire方法来设置时间
   - 如果没有设置时间，就是永不过期： ttl key  输出 -1
   - 如果设置了过期时间，又想让缓存永不过期，使用 persist key

2. 采用LRU算法动态将不用的数据删除

   内存管理的一种页面置换算法，对于在内存中但又不用的数据块（内存块）叫做LRU，操作系统会根据哪些数据属于LRU而将其移除内存而腾出空间加载另外的数据。

   - **volatile-lru** ：默认策略，只对设置过期时间的key进行LRU算法删除

   - **allkeys-lru** ：删除不经常使用的key

   - volatile-random ：随机删除即将过期的key

   - allkeys-random ：随机删除一个key

   - volatile-ttl ：删除即将过期的key

   - noeviction ：不过期，写操作返回报错
   - volatile-lfu：从所有配置了过期时间的键中驱逐使用频率最少的键
   - allkeys-lfu：从所有键中驱逐使用频率最少的键

