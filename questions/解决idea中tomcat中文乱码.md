# 解决idea中tomcat中文乱码

Help -> Edit Custom VM options -> 在idea64.exe.vmoptions中新增一行：

```
-Dfile.encoding=UTF-8
```



