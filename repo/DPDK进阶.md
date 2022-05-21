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
