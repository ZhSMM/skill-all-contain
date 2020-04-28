# redis关闭

- 断电、非正常关闭：容易造成数据丢失

  ```
  查询redis进程：
  ps -ef |grep -i redis
  
  根据redis的id kill进程：
  kill -9 PID
  ```

- 正常关闭：数据保存

  通过客户端进行shutdown，会持久化数据

