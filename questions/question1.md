### VMware Workstation 与 Device/Credential Guard 不兼容!

原文链接：[https://blog.csdn.net/luckysign/article/details/101915064](https://blog.csdn.net/luckysign/article/details/101915064)

打开的时候提示VMware Workstation 与 Device/Credential Guard 不兼容，且在禁用Device/Credential Guard后才能运行VMware，该怎么办呢，本文就给大家分享一下Win10系统下提示VMware与Device/Credential Guard不兼容的具体解决方法。
原因分析：

​	VMware和Hyper-V不兼容导致。

解决方法：

- Win10专业版解决方法：

  1、控制面板—程序——打开或关闭Windows功能，取消勾选Hyper-V，确定禁用Hyper-V服务。
  2、之后重新启动计算机，再运行VM虚拟机即可。

- Win10家庭版解决方法：

  1、按下WIN+R打开运行，然后输入services.msc回车；
  2、在服务中找到 HV主机服务，双击打开设置为禁用；

  3、windowns键+X  再打开Windows PowerShell（管理员）；

  4、运行命令：`bcdedit /set hypervisorlaunchtype off` ；

  5、重启windowns10系统；

