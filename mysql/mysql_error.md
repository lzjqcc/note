## MySQL错误日志查看

之前登录MySQL出现Error200错误，真是不知所措。

想要知道ubuntu系统下MySQL的错误日志文件在什么地方首先得查看mysql.cnf文件

![选区_016](/home/li/图片/选区_016.png)	

找到mysql.cnf文件后敲入以下命令显示文件内容

```
cat mysql.cnf
```

![选区_017](/home/li/图片/选区_017.png)	

然后打开该文件,然后找中ERROR标记的那一行

![选区_018](/home/li/图片/选区_018.png)

这里描述了unknow variable 'default-character-set=utf-8'  而这一行恰巧是我之前为了解决hibernate 添加中文乱码在**mysql.cnf**文件写上的

如此看来要删掉一行。然后重启电脑一切都好了。

![选区_019](/home/li/图片/选区_019.png)



