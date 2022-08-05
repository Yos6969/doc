[toc]

# 序-docker

docker和虚拟机不同，虚拟机使用hypervisor技术，虚拟出虚拟硬件后安装了客户操作系统，用户的机器运行在这个虚拟的机器。docker使用Namespcae做了隔离，作为一个障眼法，docker中启动的进程无法看到在他之前的进程，只能看到指定内容，但容器化后的用户依然是宿主机上的普通进程，这也意味着docker的隔离并不彻底。

其次，Linux内核中的很多资源和对象是不能被Namespace化的。同时容器里的进程和宿主机上的其他进程还是存在一个竞争关系，表面上这个进程运行在“容器”中，实际还是会存在资源被占用的问题，这不是一个沙盒应该表现出的行为。因此Linux内核中，还有一个Cgroups的功能，用来限制一个进程组能使用的资源上限，包括CPU  内存  磁盘  网络带宽等

Linux使用chroot命令，改变进程的根目录位置，达到修改文件视图的目的，一般还会往指定的路径下拷贝根目录文件，看起来像一个单独的文件系统

所以，Docker项目的核心原理就是为待创建的用户进程

1. 启用Linux Namespace配置
2. 设置指定的Cgroups参数
3. 切换进程的根目录（挂载文件）

需要明确的是，这个root文件系统只包含文件配置和目录，不包括操作系统内核。如果你的应用程序需要配置内核参数、加载额外的内核模块，以及跟内核直接交互，就需要注意了：这些操作和依赖的对象都是宿主机操作系统的内核，牵一发而动全身

# k8s

## 核心能力与项目定位

以统一的方式抽象底层基础设施能力，定义任务编排的各种关系，将这些抽象以声明式API的方式对外暴露。

- 对于频繁交互访问的容器，将他们划分在同一个Pod，Pod里的容器共享同一个Network Namespace
- 一
- 些常见的访问关系，如数据库和web应用，为了容灾性，不部署在同一机器，而是提供一种service服务。service和Pod绑定，Service服务声明的IP是固定不变的，Service服务的主要作用就是作为Pod的代理入口(Portal)。这样，对于web应用，只需要关心数据库Pod的service信息。而真正的IP地址由Kubernetes维护。

# docker

docker安装

https://cloud.tencent.com/developer/article/1701451

搜索镜像

```
docker search 
```

进入镜像

```
docker run -it -p  0.0.0.0:830:830 -v 宿主路径/容器内挂载路径   sysrepo/sysrepo-netopeer2 /bin/bash
docker run -d -p 0.0.0.0:830:830 -v /root/module/:/root/module netopeer2-server 
```

退出容器

```
exit
```

[ctrl + P] [ctrl + Q]退出而不终止容器运行

修改镜像

```
docker commit 0b2616b0e5a8 xxxx：test
```

查看运行镜像

```
docker ps
```

进入运行镜像

```
docker exec  -it  xxx /bin/bash
```

删除指定容器

```
docker rm -f <containerid>
```



docker exec 



更多配置

-d, --detach=false， 指定容器运行于前台还是后台，默认为false
-i, --interactive=false， 打开STDIN，用于控制台交互
-t, --tty=false， 分配tty设备，该可以支持终端登录，默认为false
-u, --user=""， 指定容器的用户
-a, --attach=[]， 登录容器（必须是以docker run -d启动的容器）
-w, --workdir=""， 指定容器的工作目录
-c, --cpu-shares=0， 设置容器CPU权重，在CPU共享场景使用
-e, --env=[]， 指定环境变量，容器中可以使用该环境变量
-m, --memory=""， 指定容器的内存上限
-P, --publish-all=false， 指定容器暴露的端口
-p, --publish=[]， 指定容器暴露的端口
-h, --hostname=""， 指定容器的主机名
-v, --volume=[]， 给容器挂载存储卷，挂载到容器的某个目录
--volumes-from=[]， 给容器挂载其他容器上的卷，挂载到容器的某个目录
--cap-add=[]， 添加权限，权限清单详见：http://linux.die.net/man/7/capabilities
--cap-drop=[]， 删除权限，权限清单详见：http://linux.die.net/man/7/capabilities
--cidfile=""， 运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法
--cpuset=""， 设置容器可以使用哪些CPU，此参数可以用来容器独占CPU
--device=[]， 添加主机设备给容器，相当于设备直通
--dns=[]， 指定容器的dns服务器
--dns-search=[]， 指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件
--entrypoint=""， 覆盖image的入口点
--env-file=[]， 指定环境变量文件，文件格式为每行一个环境变量
--expose=[]， 指定容器暴露的端口，即修改镜像的暴露端口
--link=[]， 指定容器间的关联，使用其他容器的IP、env等信息
--lxc-conf=[]， 指定容器的配置文件，只有在指定--exec-driver=lxc时使用
--name=""， 指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字
--net="bridge"， 容器网络设置:
bridge 使用docker daemon指定的网桥
host //容器使用主机的网络
container:NAME_or_ID >//使用其他容器的网路，共享IP和PORT等网络资源
none 容器使用自己的网络（类似--net=bridge），但是不进行配置
--privileged=false， 指定容器是否为特权容器，特权容器拥有所有的capabilities
--restart="no"， 指定容器停止后的重启策略:
no：容器退出时不重启
on-failure：容器故障退出（返回值非零）时重启
always：容器退出时总是重启
--rm， 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)
--sig-proxy=true， 设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理

#  k8s集群
