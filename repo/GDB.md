# 基础

编译一个测试程序，-g表示可以调试，命令如下：

```BASH
gcc -g test.c -o test
```

启动gdb，命令如下：

```bash
gdb test 
gdb -q test //表示不打印gdb版本信息，界面较为干净；
```

附加界面显示

```
gdb -q -tui
```

![在这里插入图片描述](.\img\gdb.jpg)

```
info break : 查看已经设置的断点
disable + 断点编号 ： 禁用断点
delete + 断点编号：删除断点
clear + 行号：删除对应行的断点
```

```
显示main.c中的main函数附近的代码: list main.c:main
显示main函数附近的代码: list main
显示main.c中的第2到20行的代码: list main.c:2,20
显示第10到20行的代码: list 10, 20
```

```
显示栈信息   bt==backtrace
frame + 栈编号 查看某一层栈的信息
up 
down   向上\下移动
```

```
打印变量
set print pretty off
```

```
set logging file <文件名>  设置输出的文件名称
set logging on  输入这个命令后，此后的调试信息将输出到指定文件
```



# 生成core文件

```
使用 ulimit -c 查看core开关，如果为0表示关闭，不会生成core文件；

使用 ulimit -c [filesize] 设置core文件大小，当最小设置为4之后才会生成core文件；

使用 ulimit -c unlimited 设置core文件大小为不限制，这是常用的做法；

如果需要开机就执行，则需要将这句命令写到 /etc/profile 等文件。

```

调试core文件

gdb [exec file] [core file]