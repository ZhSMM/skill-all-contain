# Spring Boot集成Jedis

Spring Boot框架继承了redis，在1.x.x版本使用的是jedis客户端，在2.x.x版本默认使用的lettuce客户端，两种客户端的区别如下：

Jedis：

- Jedis 是直连模式，在多个线程间共享一个 Jedis 实例时是线程不安全的；
- 如果想要在多线程环境下使用 Jedis，需要使用连接池；
- 每个线程都去拿自己的 Jedis 实例，当连接数量增多时，物理连接成本就较高了；

Lettuce：

- Lettuce的连接是基于Netty的，连接实例可以在多个线程间共享；所以，一个多线程的应用可以使用同一个连接实例，而不用担心并发线程的数量；当然这个也是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例。
- 通过异步的方式可以让我们更好的利用系统资源，而不用浪费线程等待网络或磁盘I/O；
- Lettuce 是基于 netty 的，netty 是一个多线程、事件驱动的 I/O 框架，所以 Lettuce 可以帮助我们充分利用异步的优势。

### Spring Boot集成Jedis

引入依赖：pom.xml

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

配置application.yml

```yml
server:
  port: 8080
spring:
  redis:
    database: 0 # Redis数据库索引，默认0
    port: 6379  # Redis服务器连接端口
    host: 192.168.107.141  # Redis服务器地址
    jedis:
      pool:
        max-idle: 6 # 最大空闲数
        max-active: 10 # 最大连接数
        min-idle: 2 # 最小空闲数
    timeout: 2000  # 连接超时
```

jedis配置：JedisConfig.class

```java
@Configuration
public class JedisConfig {
    private Logger logger= LoggerFactory.getLogger(JedisConfig.class);

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Value("${spring.redis.timeout}")
    private int timeout;

    @Value("${spring.redis.jedis.pool.min-idle}")
    private int minIdle;

    @Value("${spring.redis.jedis.pool.max-active}")
    private int maxActive;

    @Value("${spring.redis.jedis.pool.max-idle}")
    private int maxIdle;

    // JedisPool连接池配置
    @Bean
    public JedisPool jedisPool(){
        JedisPoolConfig jedisPoolConfig=new JedisPoolConfig();
        jedisPoolConfig.setMaxIdle(maxIdle);
        jedisPoolConfig.setMinIdle(minIdle);
        jedisPoolConfig.setMaxTotal(maxActive);

        JedisPool jedisPool=new JedisPool(jedisPoolConfig,host,port,timeout,null);

        logger.info("JedisPool连接成功："+host+"\t"+port);

        return jedisPool;
    }
}
```

jedis工具类：JedisUtil

```java
@Component
public class JedisUtil {
    @Autowired
    private JedisPool jedisPool;

    /**
     * 获取jedis资源
     * @return
     */
    public Jedis getJedis(){
        return jedisPool.getResource();
    }

    /**
     * 释放jedis连接
     * @param jedis
     */
    public void close(Jedis jedis){
        if (jedis!=null){
            jedis.close();
        }
    }
    ........................................
}
```

测试类：User.class  UserService  UserServiceImpl

```java
// User实体类，必须实现Serializable接口
@Data
public class User implements Serializable {
    private String id;
    private String name;
    private Integer age;
}
// UserService
public interface UserService {
    /**
     * Redis String 类型
     * 需求：用户输入一个key
     * 先判断Redis中是否存在该数据
     *    如果存在，在Redis中查询数据并返回
     *    如果不存在，在MySql数据库中查询，将结果赋给Redis，并返回
     */
    String getString(String key);

    /**
     * 测试String类型
     * 需求：用户输入一个redis数据，该key的有效期为28小时
     */
    void expireStr(String key, String value);

    /**
     * Hash类型演示：
     * 存一个对象：
     *   用户在前端传入一个ID编号：
     *   根据ID查询用户的对象信息
     *   先在redis中查询，存在直接返回用户结果，并返回
     *   如果redis中不存在，在mysql中查询并写入redis
     */
    User getUser(String id);
}
// UserServiceImpl
@Service
@Log //等效于 private Logger logger=LoggerFactory.getLogger(JedisConfig.class);
public class UserServiceImpl implements UserService {

    @Autowired
    private JedisPool jedisPool;   // JedisPool：Jedis连接池

    @Autowired
    private JedisUtil jedisUtil;

    @Override
    public String getString(String key) {
        // 得到Jedis对象
        Jedis jedis = jedisPool.getResource();
        String ans=null;
        // 判断key是否存在Redis
        if (jedis.exists(key)){
            ans = jedis.get(key);
            log.info("查询redis中的数据:"+ans);
        }else {
            ans="MySQL中存储的值";
            log.info("查询MySQL中的数据："+ans);
            jedis.set(key,ans);
        }
        // 关闭连接
        jedis.close();
        return ans;
    }

    @Override
    public void expireStr(String key, String value) {
        Jedis jedis = jedisUtil.getJedis();

        jedis.set(key,value);
        jedis.expire(key,20);

        log.info(key+"\t设置值："+value+"\t"+20+"s");
        jedisUtil.close(jedis);
    }

    @Override
    public User getUser(String id) {
        String key="user:"+id;  // 实体类名：id
        User user=new User();
        // 1. 得到jedis对象
        Jedis jedis = jedisUtil.getJedis();
        // 2. 业务逻辑判断
        if (jedis.exists(key)){
            Map<String, String> map = jedis.hgetAll(key);
            user.setName(map.get("name"));
            user.setId(map.get("id"));
            user.setAge(Integer.parseInt(map.get("age")));
            log.info("在redis中查询数据："+user);
        }else{
            user.setAge(30);
            user.setId(id);
            user.setName("张三");
            log.info("查询mysql数据库："+user);
            // 存入redis
            Map<String,String> map=new HashMap<>();
            map.put("id",user.getId());
            map.put("name", user.getName());
            map.put("age", user.getAge()+"");
            jedis.hmset(key, map);
            log.info("往Redis中存入数据！");
        }
        // 3. 关闭jedis连接
        jedisUtil.close(jedis);
        return user;
    }
}
```

