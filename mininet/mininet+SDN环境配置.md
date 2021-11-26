# 环境搭建

| 主机环境      | 虚拟化环境             | 终端        |
| --------- | ----------------- | --------- |
| Windows10 | vmwareworkstation | MobaXterm |

最开始准备在docker+wsl2中配置仿真环境，因为移植比较方便，有现成的镜像可以使用，wsl启动比虚拟机快太多，开销也小。

研究了两天，发现WSL2内核的 module加载会出现问题，WSL2以内建模块的方式编译内核，而mininet镜像需要加载模块，这样就需要重新编译kernel，完全按照https://zhuanlan.zhihu.com/p/138933513的步骤在WSL2搭建SDN环境，重新编译内核，又发现一个严重的问题：openvswitch编译为模块，在最新版的wsl2中行不通(版本太高了)，放弃在wsl2中搭建mininet

最后按照官方的建议<https://github.com/mininet/openflow-tutorial/wiki/Installing-Required-Software>，在虚拟机中搭建环境，步骤大致一样，除了没有使用官方建议的Putty+XMing来搭建X11图形化界面，因为mininet运行以下命令会报错

```
sudo wireshark&
```



XMing的server似乎不支持wireshark的X11传输，改用另一个控制终端MobaXterm--支持X11转发，自带SSH，能成功运行wireshark



# 运行测试

```
sudo mn --topo single,3 --mac --switch ovsk --controller remote
```

![1](.\img\1.png)



![2](.\img\2.png)



mininet测试：

1. 通过python脚本创建拓扑，一些典型的拓扑在topology zoo（[http://www.topology-zoo.org/](http://www.topology-zoo.org/)）可以找到
2. 设置链路参数：延迟、带宽、丢包率、最大队列数目
3. 主机配置：IP  路由   
4. 交换机配置：mininet可创建传统交换机和openflow交换机（通过ovs-ofctl创建流表项）





需要测试的功能：

1. 交换机多控制器连接：一个交换机或许需要连接多个控制器
2. 多控制器网络：定义了多个交换机在不同的网段，并使其分别连接到不同的控制器
3. 网络状态收集脚本：sdn控制器中能收到域中交换机的包信息，需要了解如何分类并处理这些信息