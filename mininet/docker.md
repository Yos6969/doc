# 设置mininet容器



```
docker run -it -link myryu:myryu --name mymininet --priviledged barbaracollignon/ubuntu-mininet /bin/bash
```

启动mininet容器，并指定了控制器

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

## gRPC 服务器端口

每个交换机绑定不同的gRPC端口，从50001开始递增。要将外部客户端（例如 SDN 控制器）连接到交换机，您必须发布相应的端口。

例如，当运行具有 3 个交换机的拓扑时：

```   
docker run --privileged --rm -it -p 50001-50003:50001-50003 opennetworking/mn-stratum --topo linear,3
```

为了更轻松地访问日志和其他文件，我们建议/tmp使用 docker run-v选项在主机系统上共享 容器内的目录 ，例如：

```
docker run ... -v /tmp/mn-stratum:/tmp ... opennetworking/mn-stratum ...
```

## 日志文件 

通过使用该选项在容器执行过程中，宿主系统stratum_bmv2下会出现一些与执行相关的文件/tmp/mn-stratum。文件包含在以 Mininet 中使用的交换机名称命名的目录中，例如 s1、s2 等。

这些文件的示例是：

s1/stratum_bmv2.log: 包含用于 switch 的stratum_bmv2 日志s1；
s1/chassis-config.txt：交换机启动时使用的机箱配置文件；
s1/grpc-port.txt：与此开关关联的 gRPC 端口；
s1/onos-netcfg.json：交换机连接ONOS SDN控制器的示例配置文件；

# 设置ryu容器

```
docker run -it -p  0.0.0.0:6633:6633  --name myryu osrg/ryu  /bin/bash
```

# 容器间连接link

You can link ubuntu-mininet to ubuntu-pox as follow :

docker run -p 6633:6633 -d -name poxctr manisha/ubuntu-pox pox/pox.py

docker run -i -t -link poxctr:poxctr -priviledged=true barbaracollignon/ubuntu-mininet /bin/bash

# 将容器导出为镜像

```
docker export name > imagename.tar
cat imageneme.tar | sudo docker import - imagename 

//docker push xxx
```



　docker的架构设计分为三个组件：一个客户端，一个REST API和一个服务器（守护进程）：

Client ：与REST API交互。主要目的是允许用户连接守护进程。
REST API：充当客户端和服务器之间的接口，实现通信。
守护进程：负责实际管理容器 - 启动，停止等。守护进程监听来自docker客户端的API请求。
守护进程与内核关系非常密切。今天在Windows中，当您运行Windows Server容器时，守护进程在Windows中运行。当您切换到Linux容器模式时，守护程序实际上在名为Moby Linux VM的虚拟机内运行。随着Docker 即将发布，您将能够并行运行Windows Server容器和Linux容器，守护进程将始终作为Windows进程运行。

然而，客户端不必与守护进程安装在同一个地方。例如，您可以在开发计算机上使用本地Docker客户端与Azure中的Docker进行通信。这使我们可以让WSL中的客户端与主机上运行的守护进程通信。

#  杂项

一开始准备在windows+wsl下的docker环境中进行仿真，认为主要有以下几点好处：

1. wsl可以很方便的连接windows的docker环境启动容器
2. windows提供了图形界面，用起来顺手
3. 使用虚拟机进行仿真，启动太慢

研究了两天，发现在wsl中使用mininet容器会出现问题，下载了多个mininet镜像，都会在添加openswitch这一步骤出错，最典型的错误： Module openvswitch not found indirectory /lib/modules/xxxxxWSL2  找了很久没发现有效的解决办法

知乎一篇文章<https://zhuanlan.zhihu.com/p/138933513>提到可能需要重新编译wsl内核，试试内核重新编译,成功！

接下来要做的事：

1. 在wsl2上安装docker  ==OK
2. 把之前的失败镜像拉下来跑==失败
3. 添加控制器 ==失败

重新编译内核的目的是为了使docker容器能够加载模块，但还是出现了问题---openvswitch服务无法连接（不同于之前的服务无法加载模块），尝试将openvswitch编译为模块，发现由于内核版本太新，无法进行此操作

最后放弃在wsl环境下安装docker