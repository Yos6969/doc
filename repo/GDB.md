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

# 多线程调试

- info thread  查看当前进程的线程
- thread ID 切换调试的线程为指定ID
- set scheduler-locking off|on|step   off-不锁定任何线程  on-只有当前被调程序执行  step-阻止其他线程在当前线程单步调试的时候抢占当前线程。只有当next、continue、util以及finish的时候，其他线程才会获得重新运行的。

调试正在运行的程序，首先得到PID

进入调试  gdb attach PID

玩明白了~

# 生成core文件

```
使用 ulimit -c 查看core开关，如果为0表示关闭，不会生成core文件；

使用 ulimit -c [filesize] 设置core文件大小，当最小设置为4之后才会生成core文件；

使用 ulimit -c unlimited 设置core文件大小为不限制，这是常用的做法；

如果需要开机就执行，则需要将这句命令写到 /etc/profile 等文件。

```

调试core文件

gdb [exec file] [core file]



# 借助工具查看内存泄漏

环境：Ubuntu 16.04  gcc 4.8以上
用-fsanitize=address选项编译和链接你的程序;
用-fno-omit-frame-pointer编译，以在错误消息中添加更好的堆栈跟踪。
增加-O1以获得更好的性能。

大概会拖慢两倍运行速度

下面以简单的测试代码leaktest.c为例

![img](.\img\leaktest.png)

![image-20220330112710901](.\img\image-20220330112710901.png)

![img](.\img\leakall.png)