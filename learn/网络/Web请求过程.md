## Web请求过程

### 互联网的网络架构

- C/S：客户端/服务端架构

- B/S：浏览器/服务器模式

  优点：

  - 客户端使用统一的浏览器；
  - 服务端基于统一的HTTP；

### 发起一个HTTP请求

>发起一个HTTP请求和建立一个Socket区别不大，只不过outputStream.write()写的二进制字节数据格式要符合HTTP。

1. 先根据浏览器地址栏的URL的域名DNS解析出IP地址；
2. 根据IP地址和默认的80端口与远程服务器建立Socket连接;
3. 然后根据URL组成一个get类型的HTTP请求, 通过outputStream.write()发送到目标服务器,服务器等待inputStream.write()返回数据 , 然后断开连接;

### HTTP解析

HTTP Header控制着用户浏览器的渲染行为和服务器的执行逻辑.

#### HTTP请求头

| 请求头          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Accept-Charset  | 接受指定客户端接受的字符集                                   |
| Accept-Encoding | 用于指定可接受的内容编码,如Accept-Encoding:gzip.default      |
| Accept-Language | 用于指定一种自然语言,如Accept-Language:zh-cn                 |
| Host            | 用于指定被请求资源的Internet主机和端口号,如Host:www.taobao.com |
| User-Agent      | 客户端将它的操作系统\浏览器和其他属性高速服务器              |
| Connection      | 当前连接是否保持,如Connection:Keep-Alive                     |

#### HTTP响应头

| 响应头           | 说明                                                  |
| ---------------- | ----------------------------------------------------- |
| Server           | 使用的服务器名称,如nginx                              |
| Content-Type     | 指明发送给接受者的实体正文媒体类型                    |
| Content-Encoding | 与请求Accept-Encoding对应, 高速服务器采用什么压缩编码 |
| Content-Language | 描述资源所用的自然语言, 与Accept-Language对应         |
| Content-Length   | 指明实体正文的长度, 用以字节方式存储的十进制数字表示  |
| Keep-Alive       | 保持连接的时间 , 如Keep-Alive:timeout=5,max=120       |

#### HTTP状态码

| 状态码 | 说明                                   |
| ------ | -------------------------------------- |
| 200    | 客户端请求成功                         |
| 302    | 临时跳转,跳转地址根据Location指定      |
| 400    | 客户端请求有语法错误, 无法被服务器识别 |
| 403    | 服务器收到请求,但拒绝提供服务          |
| 404    | 请求的资源不存在                       |
| 500    | 服务器发生不可预期的错误               |

### 浏览器缓存机制

浏览器缓存属于一个比较复杂的机制 , 当浏览一个页面出错时, 可以考虑是否是浏览器做了缓存 , 可以使用CTRL + F5 请求重新请求页面，在重新请求的请求头中，新增两个请求头：

Cache-Control：no-cache 和 Pragma：no-cache 。

请求头的作用：

1. Cache-Control / Pragam：用于指定所用缓存机制在整个请求/响应链中必须服从的指令。不仅可以控制浏览器，还可以控制和HTTP相关的和缓存或代理服务器。

   **可选参数：**

   - Public ：所有内容都将被缓存，在响应头中设置；
   - Private：内容只缓存到私有缓存中，在响应头中设置；
   - no-cache：所有内容均不会被缓存，在请求头和响应头中设置；
   - no-store：所有内容都不会缓存到缓存或Internet临时文件中，在响应头中设置；
   - must-revalidate/proxy-revalidation：如果缓存的内容失效，请求必须发送到服务器/代理以进行重新验证，在请求头设置；
   - max-age=xxx：缓存内容在xxx秒后失效，只能在HTTP 1.1中可以，和Last-Modified一起使用时优先级最高，在响应头中设置；

2. Expires：通用格式：`Expires:Sun, 1 Jan 2000 01:00:00 GMT`，后面跟着一个日期和时间，超过这个时间后缓存内容失效。

3. Last-Modified/Etag：

   - Last-Modified字段用于表示一个服务器上的资源的最后修改时间，资源可以是静态（静态内容自动加上Last-Modified字段）或者动态的内容（如Servlet提供了一个getLastModified方法用于检测某个动态内容是否已经更新），通过这个最后修改时间可以判断当前请求的资源是否为最新的。
   - 一般服务器在响应头返回一个Last-Modified 字段，告诉浏览器这个页面的最后修改时间，浏览器再次请求时在请求头中增加If-Modified-Since字段，询问当前缓存的页面是否为最新，如是服务器返回 304 ，服务器不会传输新数据。
   - Etag字段，让服务端给每个页面分配一个唯一的编号，通过这个编号区分页面是否为最新。

### DNS域名解析

在浏览器地址栏中输入网址后DNS解析过程：

1. 浏览器DNS缓存（缓存时间通过TTL属性设置），有就直接结束解析
2. 用户操作系统缓存
   - Linux：配置文件`/etc/hosts`
   - Windows：配置文件`C:\Windows\System32\drivers\etc\hosts`
3. 域名服务器DNS（本地区域名服务器LDNS）：
   - Linux：配置文件`/etc/resolv.conf`
   - Windows：命令ipconfig /all
4. Root Server域名服务器请求解析
5. 根域名服务器返回给本地域名服务器一个所查询域的主域名服务器（gTLD Server）地址。gTLD国际顶级域名服务器，如.com、.cn等，全球13台左右
6. LDNS 再向上一步返回的gTLD服务器发送请求
7. gTLD返回此域名对应的Name Server域名服务器的地址
8. Name Server域名服务器查询存储的域名和IP的映射关系表，得到目标IP记录，连同一个TTL值返回给DNS Server
9. LDNS缓存此域名与IP映射，缓存时间由TTL控制
10. LDNS返回解析结果给用户

#### 跟踪域名解析过程

- Windows和Linux均可以使用nslookup命令查询域名解析结果
- Linux还可以使用 dig 跟踪查询DNS的解析过程：`dig [url] +trace`可以跟踪域名解析过程；

#### 清除域名的缓存

- DNS域名解析后会缓存解析结果，主要存放于：

  - Local DNS Server
  - 用户本地机器
  - 以上两个缓存受TTL和本地缓存大小控制

- LDNS缓存受TTL控制，很难介入，本机缓存可以控制：

  - Windows：ipconfig /flushdns 刷新缓存
  - Linux：`/etc/init.d/nscd restart`清除缓存

- Java应用中，JVM也会缓存DNS的解析结果，这个缓存是再InetAddress类中完成，此缓存有两种策略：

  - 正确解析结果缓存
  - 失败解析结果缓存

  缓存时间由两个配置项控制，配置项在%JAVA_HOME%\lib\security\java.security文件中配置，配置项分别是：

  - networkaddress.cache.ttl  ：默认值 -1 ，永不失效
  - networkaddress.cache.negative.ttl：默认值10 ，缓存10s

  修改：

  - 直接修改配置文件java.security默认值
  - 在Java启动参数添加 -Dsun.net.inetaddr.ttl = xxx来修改默认值
  - 通过InetAddress类动态修改

  注意：

  使用InetAddress类解析域名，必须是单例模式，否则会有很严重的性能问题。如果每次都创建InetAddress实例，则每次都要进行一次完整的域名解析，非常耗时。

#### 域名解析方式

- A记录：A代表Address，用来指定域名对应的IP地址，A记录可以将多个域名解析到一个IP，不能将一个域名解析到多个IP地址
- MX记录：Mail Exchange，可以将某个域名下的邮件服务器指向自己的Mail Server
- CNAME记录：Canonical Name，别名解析，即为一个域名设置一个或多个别名
- NS记录：为某个域名指定DNS解析服务器，也就是这个域名由指定的IP地址的DNS服务器区解析
- TXT记录：为某个主机名或域名设置说明

