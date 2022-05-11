# VXLAN

## 概念

VXLAN分为静态VXLAN和动态VXLAN，动态的VXLAN由evpn生成。

Vtep：VTEP是VXLAN隧道端点，封装在NVE中，用于VXLAN报文的封装 和解封装。

VNI: vxlan network ID (类似vlan的ID一样)；

BD：BD是VXLAN网络中转发数据报文的二层广播域。 在VXLAN网络中，将VNI映射到广播域BD，BD成为VXLAN网络转 发数据报文的实体。

vBDif:基于BD创建的三层逻辑接口。通过VBDIF接口配置IP地址可实现 不同网段的VXLAN间，及VXLAN和非VXLAN的通信，也可实现二 层网络接入三层网络。

NVE：设备运行了vxlan就是NVE


##引入

https://www.bilibili.com/video/BV1oK4y1L795/

![image-20220510105809400](.\img\image-20220510105809400.png)

云数据中心大量使用虚拟机，所以东西向流量占比高，同时会有虚拟机迁移的业务，引入VXLAN做大二层网络转发

<img src=".\img\image-20220510110243804.png" alt="image-20220510110243804" style="zoom:70%;" />

传统数据中心存在的问题：

- 存在环路，需要部署stp协议破环
- 不支持ECMP，链路利用率低
- 收敛慢
- 支持网络设备数量少，一般小于100台

<img src=".\img\image-20220510111213837.png" alt="image-20220510111213837" style="zoom:75%;" />

##二层转发

![image-20220510111716890](.\img\image-20220510111716890.png)

<img src=".\img\image-20220510112242444.png" alt="image-20220510112242444" style="zoom:50%;" />

TOR为接入层交换机。每台设备上可以创建很多个ID（类似于vlan一样），所有vni共享一个vtep，每个vni有自己的一个广播域，每个vni对应一个BD

左侧VM1和VM2因为接入同一个OVS交换机，直接根据OVS的MAC地址表进行转发

右侧VM3和VM4因为在同一TOR交换机, 相同BD域，根据TOR的MAC地址表转发

下侧VM1和VM6在不同TOR，但在同一个VLAN，会进行VXLAN封装,到达对端后解封装

## 三层转发

跨vni的通信，需要一个网关进行转发，网关中创建vbdif，识别VXLAN报文进行转发

下图VM1与VM2通信

![img](.\img\watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5paw5omL56yU6K6wMDg=,size_10,color_FFFFFF,t_70,g_se,x_16)

<img src=".\img\image-20220510120055772.png" alt="image-20220510120055772" style="zoom:50%;" />

![image-20220510155304115](.\img\image-20220510155304115.png)

![image-20220510155322812](.\img\image-20220510155322812.png)

![image-20220510155332571](.\img\image-20220510155332571.png)

需要集中式网关进行转发，BDIF接口进行VXLAN封装

## EVPN

EVPN是一种二层vpn技术，控制平面采用MP-BGP通告EVPN路由信息

- 简化配置：通过MP-BGP实现VTEP自动发现、VXLAN隧道自动建立、VXLAN隧道与VXLAN自动关联，无需用户手动配置
- 转控分离：控制平面负责发布路由信息，数据平面负责转发报文，分工明确

