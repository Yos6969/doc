[toc]

# 文件挂载

指的就是将设备文件中的顶级目录连接到 Linux 根目录下的某一目录（最好是空目录），访问此目录就等同于访问设备文件。

Linux 系统中“一切皆文件”，所有文件都放置在以根目录为树根的树形目录结构中。在 inux 看来，任何硬件设备也都是文件，它们各有自己的一套文件系统（文件目录结构）。

> 因此产生的问题是，当在 Linux 系统中使用这些硬件设备时，只有将Linux本身的文件目录与硬件设备的文件目录合二为一，硬件设备才能为我们所用。合二为一的过程称为“挂载”。

如果不挂载，通过Linux系统中的图形界面系统可以查看找到硬件设备，但命令行方式无法找到。

并不是根目录下任何一个目录都可以作为挂载点，由于挂载操作会使得原有目录中文件被隐藏，因此根目录以及系统原有目录都不要作为挂载点，会造成系统异常甚至崩溃，挂载点最好是新建的空目录。

##“挂载点”的目录要求：

- 目录事先存在，可以用mkdir命令新建目录
- 挂载点目录不可被其他进程使用到
- 挂载点下原有文件将被隐藏

##mount命令格式

```c++
//bash  mount [-t vfstype] [-o options] [设备名称] [挂载点]
int mount(const char *source, const char *target,
                 const char *filesystemtype, unsigned long mountflags,
                 const void *data);
/*
source: 文件系统所在设备名称（或网络文件系统的remote挂载点等）
target: 要挂载到的位置，一般是目录名。
filesystemtype: 文件系统名称。
mountflags: 文件系统通用挂载选项。
data: 文件系统特用挂载选项
*/
```

# screen

通过SSH远程登录到 Linux 服务器，运行耗时较长的程序，推荐使用screen命令：

- screen -S name ：启动一个名字为name的screen
- screen -S name -X quit ：删除某个session
- screen -ls ：列出所有的screen
- screen -r name或者id：连接回到某个screen
- ctrl + a + d ：暂时断开当前screen会话
- ctrl + a + c ：screen 在该会话内生成一个新的窗口并切换到该窗口

docker run -d -p 0.0.0.0:830 -v /home/yos/module/:/module netopeer2-server -d 

# ps 

ps -aux  和 ps -a

pstree -a 显示进程树

# ldconfig

vim /etc/ld.so.conf 直接加一行，增加库搜索目录

然后 ldconfig加载

# 开启服务

service xrdp start

# 查看系统参数

```
查看CPU信息：
	ps：显示按照按照消耗CPU前10排序的进程
	top：任务、CPU状态、内存状态、各进程的状态监控
查看内存信息：
	ps
	top
	pmap：查看进程的内存状态，以及内存映射
查看磁盘IO信息
	iotop
	iostat
查看端口信息
	netstat
查看大小端和cpu参数
	lscpu
查看当前缓存中存在的动态链接库
	ldconfig -p 
查看磁盘信息
	df -h
查看网络流量
	iftop
```

# 
