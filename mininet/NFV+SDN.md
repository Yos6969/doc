# LTE移动核心网虚拟化

## 定义

EPC是LTE中的核心网，提供用户追踪、移动性管理和会话管理等功能，用户设备通过接入网连接到EPC。现网的EPC网络都是专用设备构成的，如果设备需要在功能、容量上做小改动，就需要更换设备。虚拟化后，可以提供更好的灵活性和动态扩容，可以实现服务创新，能有效降低成本。

![1](.\img\1.jpg)

![2](.\img\2.jpg)

虚拟网络功能可独立地根据自己的具体资源需求进行伸缩，每个具体的VNF需要的资源也不同，处理数据平面的VNFS要比只处理信令的VNFs消耗更多的NFV基础设施资源

再者，基于NFV的虚拟化移动核心网需要与现有的移动核心网络共存，主要考虑两种方案：

1. 核心网络部分组件实现虚拟化
2. 虚拟化和非虚拟化的移动核心网络的共存

![3](.\img\3.jpg)

核心网虚拟化面临以下的挑战：

1. 资源伸缩：虚拟化EPC和IMS网络资源的向上、向下伸缩
2. 虚拟化对服务的透明性：运行服务时不需要知道网络功能是否被虚拟化
3. 虚拟化对网络控制的管理的透明性：网络控制和管理平面不需要知道网络功能是否被虚拟化
4. 状态维护：在网络功能复制、迁移以及资源伸缩时，网络以及网络功能的状态要保持正常
5. 检测|故障诊断|恢复：提供适当的机制用于虚拟化组件的检查与恢复
6. 服务可用性：与非虚拟化的网络一样，虚拟化的移动核心网提供相同水平的服务可用性
7. 流量控制分离机制：提供虚拟化和非虚拟化核心网中数据及管理流量的认真与分离
8. 减少对现有非虚拟化网络功能的影响



5G组网功能元素可分文四个层次：

1. 中心级：以控制、管理和调度职能为核心，例如虚拟化功能编排、广域数据中心互联等，可按需部署于全国节点，实现网络总体的监控和维护
2. 汇聚级：主要包括控制面网络功能，例如移动性管理、会话管理、用户数据和策略等，可按需部署于省份一级网络
3. 区域级：主要功能包括数据面网关功能，重点承载业务数据流，可部署于地市一级。移动边缘计算功能、业务链功能和部分控制面网络功能也可以下沉到这一级
4. 接入级：包含无线接入网的CU和DU功能，CU可部署在回传网络的接入层或汇聚层；DU部署在用户近端。CU和DU间通过增强的低延时传输网络实现多点协作化功能，支持分离或一体化站点的灵活组网

![4](.\img\4.jpg)

![5](.\img\5.jpg)