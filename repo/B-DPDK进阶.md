[toc]

编译主目录下example中的文件

```c
cd helloworld
make
```

显示

```
Makefile:14: *** "no installation of DPDK found".  Stop.
```

进入makefile发现是pkg-config命令出错，没有找到dpdk的安装文件

```bash
[yoscentos@localhost skeleton]$ pkg-config libdpdk --libs
Package libdpdk was not found in the pkg-config search path.
Perhaps you should add the directory containing `libdpdk.pc'
to the PKG_CONFIG_PATH environment variable
No package 'libdpdk' found
```

指定一下搜索目录

```
export PKG_CONFIG_PATH="/usr/local/lib64/pkgconfig/"
```

ok

```bash
[yoscentos@localhost skeleton]$ make
ln -sf basicfwd-shared build/basicfwd
```



# HELLO_WORLD

```bash
build/helloworld -l 0-3 -n 2
# 有以下输出既是成功
# hello from core 1
# hello from core 2
# hello from core 3
# hello from core 0
```

以下参数都是比较常用的

- `c COREMASK`: 要运行的内核的十六进制掩码。注意，平台之间编号可能不同，需要事先确定。
- `l x-y`: 指定处理器核心运行，一般与 `c` 不共用。
- `n NUM`: 每个处理器插槽的内存通道数目。
- `b <domain:bus:devid.func>`: 端口黑名单，避免EAL使用指定的PCI设备。
- a: 仅使用指定的以太网设备。使用逗号分隔 `[domain:]bus:devid.func` 值，不能与 `b` 选项一起使用。
- `-socket-mem`: 从特定插槽上的hugepage分配内存。
- `m MB`: 内存从hugepage分配，不管处理器插槽。建议使用 `-socket-mem` 而非这个选项。
- `r NUM`: 内存数量。
- `v`: 显示启动时的版本信息。
- `-huge-dir`: 挂载hugetlbfs的目录。
- `-file-prefix`: 用于hugepage文件名的前缀文本。
- `-proc-type`: 程序实例的类型。
- `-xen-dom0`: 支持在Xen Domain0上运行，但不具有hugetlbfs的程序。
- `-vmware-tsc-map`: 使用VMware TSC 映射而不是本地RDTSC。
- `-base-virtaddr`: 指定基本虚拟地址。
- `-vfio-intr`: 指定要由VFIO使用的中断类型。(如果不支持VFIO，则配置无效)。

[helloworld](../dpdk-source/helloworld.md)

1.初始化基础运行环境

```c
int rte_eal_init(int argc, char **argv);
```

- 配置初始化
- 内存初始化
- 内存池初始化
- 队列初始化
- 告警初始化
- 中断初始化
- PCI初始化
- 定时器初始化
- 检测内存本地化(NUMA)
- 插件初始化
- 主线程初始化
- 轮询设备初始化
- 建立主从线程通道
- 将从线程设置在等待模式
- PCI设备的探测与初始化

2.多核运行初始化

```c
int rte_eal_remote_launch(int (*f)(void *), void *arg, unsigned slave_id)
```

第一个参数为从线程，是被征召的线程

第二个参数是传给从线程的参数

第三个参数是指定的逻辑核，从线程执行在这个core上

# Skeleton

一个简单的报文收发实例，对收入报文不做任何处理直接发送，从网口抓数据包转发到另一个网口

dpdk需要绑定网卡，在dpdk-21.11.1版本，使用dpdk-kmod

https://www.cxymm.net/article/qiqicao123456/108473455 //虚拟网卡使用kmod的坑

```bash
git clone https://github.com/atsgen/dpdk-kmod.git
cd dpdk-kmod/scripts
sudo sh install.sh 
sudo modprobe igb_uio
sudo modprobe uio
```



```bash
./usertools/dpdk-devbind.py --status
#查看绑定状态
ifconfig ens34 down#先关闭接口#systemctl restart network||reboot 重新启用
sudo ./usertools/dpdk-devbind.py --bind=igb_uio ens34
sudo ./usertools/dpdk-devbind.py --bind=igb_uio ens35
```

[skeleton](../dpdk-source/skeleton.md)

```bash
#网卡配置https://dev-tang.com/post/2018/07/centos-network.html
cd /etc/sysconfig/network-scripts
vim ifcfg-eno1
```

修改配置项,只贴出需要修改或增加的地方

```
BOOTPROTO=static # ip改成静态获取

IPADDR=192.168.0.212 # 固定ip
NETMASK=255.255.255.0 # 子网掩码
GATEWAY=192.168.0.1 # 网关地址
DNS1=192.168.0.1 # DNS

ONBOOT=yes # 开启启动
```

# l2fwd

[l2fwd](../dpdk-source/l2fwd.md)

![img](\img\5971286-43474dabaf94f681.png)

#	testpmd

在build/app目录下有一个testpmd的程序，用它来检查使用DPDK的不同网络设备的性能和功能

## TestPMD的配置示例

为了展示如何使用TestPMD，我们会考虑两个典型的硬件设置。
如图1所示，第一个配置，TestPMD应用程序把两个以太网口连接到外部的流量发生器。这样用户可以在不同的网络工作负载下测试吞吐量和功能。
![请输入图片名称](\img\dpdk0830-21.webp.jpg)
图1. 设置1——使用外部流量发生器

第二个设置，TestPMD应用程序把两个以太网端口连成环回模式。 这样用户可以在没有外部流量发生器的情况下检查网络设备的接收和传输功能。
![请输入图片名称](\img\dpdk0830-22.jpg)
图2.设置2 -环回模式的TestPMD

## 转发模式

estPMD可以使用如下几种不同的转发模式。
**输入/输出模式（Input/output mode）**：此模式通常称为IO模式，是最常用的转发模式，也是TestPMD启动时的默认模式。 在IO模式下，CPU内核从一个端口接收数据包（Rx），并将其发送到另一个端口（Tx）。 如果需要的话，一个端口可同时用于接收和发送。
**收包模式（Rx-only mode）**：在此模式下，应用程序会轮询Rx端口的数据包，然后直接释放而不发送。 它以这种方式充当数据包接收器。
**发包模式（Tx-only mode）**：在此模式下，应用程序生成64字节的IP数据包，并从Tx端口发送出去。 它不接收数据包，仅作为数据包源。
后两种模式（收包模式和发包模式）对于单独检查收包或者发包非常有用。
除了这三种模式，TestPMD文档中还介绍了其他一些转发模式。

```bash
sudo ./dpdk-testpmd -l 0-3 -n 4 -- -i --nb-ports=2 --nb-cores=2
```

更多命令查看https://dpdk-docs.readthedocs.io/en/latest/testpmd_app_ug/run_app.html#eal-command-line-options

# 巨页操作

2M巨页和1G巨页二选一

2m可以在系统启动后追加，使用echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

https://blog.csdn.net/m0_37986160/article/details/121661297

## 2M巨页

**修改内核参数 **

```bash
sudo vim /etc/default/grub
在GRUB_CMDLINE_LINUX参数后追加:
hugepages=1024
#1024表示有1024个巨页
如果需要开启immon功能需要追加：
intel_iommu=on iommu=pt
```

**重新生成引导配置**

```bash
 sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

**重启验证**

```bash
sudo reboot
cat /proc/meminfo |grep -i HugePages
验证iommu是否成功：
dmesg | grep DMAR
```

## 1G巨页

**修改内核参数**

```bash
sudo vim /etc/default/grub

在GRUB_CMDLINE_LINUX参数后追加:
default_hugepagesz=1G hugepagesz=1G hugepages=4 
#GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet default_hugepagesz=1G hugepagesz=1G hugepages=4"
如果需要开启intel_iommu功能需要追加：
intel_iommu=on iommu=pt
```

其余步骤参考2M页设置

## 挂载巨页

```bash
# 将大页内存分配给DPDK
mkdir /dev/hugepages
mount -t hugetlbfs pagesize=1GB /dev/hugepages

# 查看 Linux 操作系统是否启动了大页内存，如果 HugePages_Total 为 0，意味着 Linux 没有设置或没有启用 Huge pages。
grep -i HugePages_Total /proc/meminfo

# 查看是否挂载了 hugetlbfs
mount | grep hugetlbfs

# 查看更详细的大页内存信息
cat /proc/meminfo | grep Huge
```

##配置启动自动挂载

```bash
sudo vim /etc/fstab

追加自动挂载：
nodev /mnt/huge hugetlbfs pagesize=1GB 0 0
```

# 抓包dpdk

网卡被DPDK接管后，内核的ifconfig，tcpdump等工具将不再适用（由mTCP中修改后的DPDK绑定的网卡，是可以通过ifconfig命令看到的），所以开发调试中，抓包就成了问题。一般对端没有使用DPDK的话，会选择在对端使用tcpdump抓包或者经过交换机，在交换机抓包。但是，如果想在DPDK的出入口抓包，或者收发两端都使用DPDK，使用Linux提供的工具直接抓包就很难实现。



pdump库是在DPDK 16.07版本引入的，提供了一个抓包调试功能。在$(RTE_SDK)/app目录下有一个dpdk-pdump工具，可以用于抓取指定接口、队列 的网络包。

## 原理

- pdump抓包采用的是 **CS** 模式，即应用进程作为server，拷贝数据包，提供给dpdk-pdump；dpdk-dump作为client，接收数据包，并dump到pcap文件中
- dpdk-pdump作为 **secondary** 进程 依附于 **primary** 进程
- primary进程中启动 server 端，初始化pdump抓包框架任务；
- dpdk-pdump进程是作为 client端 向primary进程发送开始/停止抓包请求；
- primary进程 拷贝 一份数据包到ring中，secondary进程从ring中读取出来，保存为pcap文件；

![img](\img\pdump-framework.png)

#DPDK轮询模式驱动

DPDK包括Gigabit、10Gigabit 及 40Gigabit 和半虚拟化IO的轮询模式驱动程序。

轮询模式驱动程序(PMD)由通过在用户空间中运行的BSD驱动提供的API组成，以配置设备及各自的队列。

此外，PMD直接访问 RX 和 TX 描述符，且不会有任何中断（链路状态更改中断除外）产生，这可以保证在用户空间应用程序中快速接收，处理和传送数据包。 本节介绍PMD的要求、设计原则和高级架构，并介绍了以太网PMD的对外通用API。

## 要求及假设条件

DPDK环境支持两种模式的数据包处理，RTC和pipeline：

- 在 *run-to-completion* 模式中，通过调用API来轮询指定端口的RX描述符以获取报文。 紧接着，在同一个core上处理报文，并通过API调用将报文放到接口的TX描述符中以发送报文。
- 在 *pipe-line* 模式中，一个core轮询一个或多个接口的RX描述符以获取报文。然后报文经由ring被其他core处理。 其他core可以继续处理报文，最终报文被放到TX描述符中以发送出去。

在同步 run-to-completion 模式中，每个逻辑和处理数据包的流程包括以下步骤：

- 通过PMD报文接收API来获取报文
- 一次性处理每个数据报文，直到转发阶段
- 通过PMD发包API将报文发送出去

相反地，在异步的pipline模式中，一些逻辑核可能专门用于接收报文，其他逻辑核用于处理前面收到的报文。 收到的数据包通过报文ring在逻辑核之间交换。 数据包收包过程包括以下步骤：

- 通过PMD收包API获取报文
- 通过数据包队列想逻辑核提供接收到的数据包

数据包处理过程包括以下步骤：

- 从数据包队列中获取数据包
- 处理接收到的数据包，直到重新发送出去

为了避免任何不必要的中断处理开销，执行环境不得使用任何异步通知机制。即便有需要，也应该尽量使用ring来引入通知信息。

在多核环境中避免锁竞争是一个关键问题。 为了解决这个问题，PMD旨在尽可能地使用每个core的私有资源。 例如，PMD每个端口维护每个core单独的传输队列。 同样的，端口的每个接收队列都被分配给单个逻辑核并由其轮询。

# EAL环境层

https://dpdk-docs.readthedocs.io/en/latest/prog_guide/env_abstraction_layer.html#linuxeal

- EAL提供的典型服务有：
  - DPDK的加载和启动：DPDK和指定的程序链接成一个独立的进程，并以某种方式加载
  - CPU亲和性和分配处理：DPDK提供机制将执行单元绑定到特定的核上，就像创建一个执行程序一样。
  - 系统内存分配：EAL实现了不同区域内存的分配，例如为设备接口提供了物理内存。
  - PCI地址抽象：EAL提供了对PCI地址空间的访问接口
  - 跟踪调试功能：日志信息，堆栈打印、异常挂起等等。
  - 公用功能：提供了标准libc不提供的自旋锁、原子计数器等。
  - CPU特征辨识：用于决定CPU运行时的一些特殊功能，决定当前CPU支持的特性，以便编译对应的二进制文件。
  - 中断处理：提供接口用于向中断注册/解注册回掉函数。
  - 告警功能：提供接口用于设置/取消指定时间环境下运行的毁掉函数。

##Linux环境下的EAL

在Linux用户空间环境，DPDK APP通过pthread库作为一个用户态程序运行。 设备的PCI信息和地址空间通过 /sys 内核接口及内核模块如uio_pci_generic或igb_uio来发现获取的。 linux内核文档中UIO描述，设备的UIO信息是在程序中用mmap重新映射的。

EAL通过对hugetlb使用mmap接口来实现物理内存的分配。这部分内存暴露给DPDK服务层，如 [Mempool Library](https://dpdk-docs.readthedocs.io/en/latest/prog_guide/mempool_lib.html#mempool-library)。

据此，DPDK服务层可以完成初始化，接着通过设置线程亲和性调用，每个执行单元将会分配给特定的逻辑核，以一个user-level等级的线程来运行。

### 内存映射和内存分配

大量连续的物理内存分配时通过hugetlbfs内核文件系统来实现的。 EAL提供了相应的接口用于申请指定名字的连续内存空间。 这个API同时会将这段连续空间的地址返回给用户程序。

### PIC访问，如网卡

EAL使用Linux内核提供的文件系统 /sys/bus/pci 来扫描PCI总线上的内容。 内核模块uio_pci_generic提供了/dev/uioX设备文件及/sys下对应的资源文件用于访问PCI设备。 DPDK特有的igb_uio模块也提供了相同的功能用于PCI设备的访问。 这两个驱动模块都用到了Linux内核提供的uio特性。

### 跟踪与调试

Glibc中提供了一些调试函数用于打印堆栈信息。 Rte_panic函数可以产生一个SIG_ABORT信号，这个信号可以触发产生core文件，可以通过gdb来加载调试。

### 日志

EAL提供了日志信息接口。 默认的，在linux 应用程序中，日志信息被发送到syslog和concole中。 当然，用户可以通过使用不同的日志机制来代替DPDK的日志功能。

### 中断

- 主线程中的用户空间中断和警告处理

EAL创建一个主线程用于轮询UIO设备描述文件以检测中断。 可以通过EAL提供的函数为特定的中断事件注册/解注册回掉函数，回掉函数在主线程中被异步调用。 EAL同时也允许像NIC中断那样定时调用回掉函数。

- RX 中断事件

PMD提供的报文收发程序并不只限制于自身轮询下执行。 为了缓解小吞吐量下轮询模式对CPU资源的浪费，暂停轮询并等待唤醒事件发生时一种有效的手段。 收包中断是这种场景的一种很好的选择，但也不是唯一的。

EAL提供了事件驱动模式相关的API。以Linux APP为例，其实现依赖于epoll技术。 每个线程可以监控一个epoll实例，而在实例中可以添加所有需要的wake-up事件文件描述符。 事件文件描述符创建并根据UIO/VFIO的说明来映射到制定的中断向量上。 对于BSD APP，可以使用kqueue来代替，但是目前尚未实现。

EAL初始化中断向量和事件文件描述符之间的映射关系，同时每个设备初始化中断向量和队列之间的映射关系， 这样，EAL实际上并不知道在指定向量上发生的中断，由设备驱动负责执行后面的映射。

RX中断由API（rte_eth_dev_rx_intr_*）来实现控制、使能、关闭。当PMD不支持时，这些API返回失败。Intr_conf.rxq标识用于打开每个设备的RX中断。
