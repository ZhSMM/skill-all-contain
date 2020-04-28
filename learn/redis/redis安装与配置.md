# redis

### 安装

- 安装gcc：`sudo yum -y install gcc automake autoconf libtool make`

- 官网下载：[中文版官网](http://www.redis.cn/)
- 解压缩：`tar zxvf redis-xx.tar.gz -C /opt` 解压到opt文件
- 编译：`cd opt/redis-5.0.5 & make`
- 指定位置安装：`make PREFIX=/usr/local/redis install`
- 启动：
  - 服务端： ./redis/bin/redis-server
  - 客户端： ./redis/bin/redis-cli -h IP地址 -p 端口   // 默认本机地址，6379端口
- 查看redis进程：ps -ef | grep -i redis

### redis配置

Redis可以在没有配置文件的情况下通过内置的配置来启动，但是这种启动方式只适用于开发和测试。合理的配置Redis的方式是提供一个Redis配置文件，这个文件通常叫做 `redis.conf`。

redis 的配置文件位于Redis安装目录下，文件名 redis.conf

命令：解压目录下的redis.conf配置文件复制到安装文件目录下

```
cp /opt/redis-5.0.5/redis.conf /usr/local/redis
```

redis.conf文件中包含了很多格式简单的指令如下：

```
keyword argument1 argument2 ... argumentN
关键字   参数1     参数2      ... 参数N
```

如下是一个配置指令的示例：

```
slaveof 127.0.0.1 6380
```

如果参数中含有空格，那么可以用双引号括起来，如下：

```
requirepass "hello world"
```

通过命令行传参：与配置文件一致，需要加 `--`

redis.conf详细配置：

```
daemonize yes #是否以后台进程运行

pidfile /var/run/redis/redis-server.pid    #pid文件位置

port 6379#监听端口

bind 127.0.0.1   #绑定地址，如外网需要连接，设置0.0.0.0

timeout 300     #连接超时时间，单位秒

loglevel notice  #日志级别，分别有：

# debug ：适用于开发和测试

# verbose ：更详细信息

# notice ：适用于生产环境

# warning ：只记录警告或错误信息

logfile /var/log/redis/redis-server.log   #日志文件位置

syslog-enabled no    #是否将日志输出到系统日志

databases 16#设置数据库数量，默认数据库为0



############### 快照方式 ###############



save 900 1    #在900s（15m）之后，至少有1个key发生变化，则快照

save 300 10   #在300s（5m）之后，至少有10个key发生变化，则快照

save 60 10000  #在60s（1m）之后，至少有1000个key发生变化，则快照

rdbcompression yes   #dump时是否压缩数据

dir /var/lib/redis   #数据库（dump.rdb）文件存放目录



############### 主从复制 ###############



slaveof <masterip> <masterport>  #主从复制使用，用于本机redis作为slave去连接主redis

masterauth <master-password>   #当master设置密码认证，slave用此选项指定master认证密码

slave-serve-stale-data yes     #当slave与master之间的连接断开或slave正在与master进行数据同步时，如果有slave请求，当设置为yes时，slave仍然响应请求，此时可能有问题，如果设置no时，slave会返回"SYNC with master in progress"错误信息。但INFO和SLAVEOF命令除外。



############### 安全 ###############



requirepass foobared   #配置redis连接认证密码



############### 限制 ###############



maxclients 128#设置最大连接数，0为不限制

maxmemory <bytes>#内存清理策略，如果达到此值，将采取以下动作：

# volatile-lru ：默认策略，只对设置过期时间的key进行LRU算法删除

# allkeys-lru ：删除不经常使用的key

# volatile-random ：随机删除即将过期的key

# allkeys-random ：随机删除一个key

# volatile-ttl ：删除即将过期的key

# noeviction ：不过期，写操作返回报错

maxmemory-policy volatile-lru#如果达到maxmemory值，采用此策略

maxmemory-samples 3   #默认随机选择3个key，从中淘汰最不经常用的



############### 附加模式 ###############



appendonly no    #AOF持久化，是否记录更新操作日志，默认redis是异步（快照）把数据写入本地磁盘

appendfilename appendonly.aof  #指定更新日志文件名

# AOF持久化三种同步策略：

# appendfsync always   #每次有数据发生变化时都会写入appendonly.aof

# appendfsync everysec  #默认方式，每秒同步一次到appendonly.aof

# appendfsync no       #不同步，数据不会持久化

no-appendfsync-on-rewrite no   #当AOF日志文件即将增长到指定百分比时，redis通过调用BGREWRITEAOF是否自动重写AOF日志文件。



############### 虚拟内存 ###############



vm-enabled no      #是否启用虚拟内存机制，虚拟内存机将数据分页存放，把很少访问的页放到swap上，内存占用多，最好关闭虚拟内存

vm-swap-file /var/lib/redis/redis.swap   #虚拟内存文件位置

vm-max-memory 0    #redis使用的最大内存上限，保护redis不会因过多使用物理内存影响性能

vm-page-size 32    #每个页面的大小为32字节

vm-pages 134217728  #设置swap文件中页面数量

vm-max-threads 4    #访问swap文件的线程数



############### 高级配置 ###############



hash-max-zipmap-entries 512   #哈希表中元素（条目）总个数不超过设定数量时，采用线性紧凑格式存储来节省空间

hash-max-zipmap-value 64     #哈希表中每个value的长度不超过多少字节时，采用线性紧凑格式存储来节省空间

list-max-ziplist-entries 512  #list数据类型多少节点以下会采用去指针的紧凑存储格式

list-max-ziplist-value 64    #list数据类型节点值大小小于多少字节会采用紧凑存储格式

set-max-intset-entries 512   #set数据类型内部数据如果全部是数值型，且包含多少节点以下会采用紧凑格式存储

activerehashing yes        #是否激活重置哈希



总结：

1、redis提供几种持久化机制：

 a). RDB持久化

 工作方式 ：根据时间的间隔将redis中数据快照（dump）到dump.rdb文件

 优势 ：备份恢复简单。RDB通过子进程完成持久化工作，相对比AOF启动效率高

 劣势 ：服务器故障会丢失几分钟内的数据

 b). AOF持久化

 工作方式 ：以日志的形式记录所有更新操作到AOF日志文件，在redis服务重新启动时会读取该日志文件来重新构建数据库，以保证启动后数据完整性。

 优势 ：AOF提供两种同步机制，一个是fsync always每次有数据变化就同步到日志文件和fsync everysec每秒同步一次到日志文件，最大限度保证数据完整性。

 劣势：日志文件相对RDB快照文件要大的多

 AOF日志重写功能 ：AOF日志文件过大，redis会自动重写AOF日志，append模式不断的将更新记录写入到老日志文件中，同时redis还会创建一个新的日志文件用于追加后续的记录。

 c). 同时应用AOF和RDB

 对于数据安全性高的场景，可同时使用AOF和RDB，这样会降低性能。

 d). 无持久化

 禁用redis服务持久化功能。

2、AOF日志文件出错后，修复方法 ：

redis-check-aof --fix appendonly.aof  #--fix参数为修复日志文件，不加则对日志检查

3、不重启redis从RDB持久化切换到AOF持久化 ：

redis-cli> CONFIG SET appendonly yes      #启用AOF

redis-cli> CONFIG SET save ""         #关闭RDB
```

