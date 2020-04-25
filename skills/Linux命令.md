# Linux命令

### 网络相关

- 查看本机IP：`ifconfig`
- 查看路由：`netstat -rn`
  - 以`0.0.0.0`开始的为默认网关
- 查看DNS服务器：`cat /etc/resolv.conf`

### 系统信息

- uname ：简单信息
  - -a ：稍微具体一点的信息
- cat  /proc/version：操作系统发行版本信息
- cat  /etc/issue：内核信息