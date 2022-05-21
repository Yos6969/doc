# 概述

DPDK的主要目标就是要为数据面快速报文处理应用提供一个简洁但是完整的框架。 用户可以通过代码来理解其中使用的一些技术，并用来构建自己的应用原型或是添加自己的协议栈。 用户也可以替换DPDK提供的原生的选项。

通过创建环境抽象层EAL，DPDK框架为每个特殊的环境创建了运行库。 这个环境抽象层是对底层架构的抽象，通过make和配置文件，在Linux用户空间编译完成。 一旦EAL库编译完成，用户可以通过链接这些库来构建自己的app。 除开环境抽象层，还有一些其他库，包括哈希算法、最长前缀匹配、环形缓冲器。 DPDK提供了一些app用例用来指导如何使用这些特性来创建自己的应用程序。

DPDK实现了run-to-complete报文处理模型，数据面处理程序在调用之前必须预先分配好所有的资源，并作为执行单元运行与逻辑核心上。 这种模型并不支持调度，且所有的设备通过轮询方式访问。 不使用中断方式的主要原因就是中断处理增加了性能开销

作为RTC模型的扩展，通过使用ring在不同core之间传递报文和消息，也可以实现报文处理的流水线模型（pipeline）。 流水线模型允许操作分阶段执行，在多核代码执行中可能更高效。

## **DPDK的优势**

DPDK应用程序是运行在用户空间上利用自身提供的数据平面库来收发数据包，绕过了Linux内核协议栈对数据包处理过程。Linux内核将DPDK应用程序看作是一个普通的用户态进程，包括它的编译、连接和加载方式和普通程序没有什么两样。

DPDK的**优势**：

减少了中断次数。

减少了内存拷贝次数。

绕过了linux的协议栈，进入用户协议栈，用户获得了协议栈的控制权，能够定制化协议栈降低复杂度

DPDK的**劣势**：

内核栈转移至用户层增加了开发成本.

低负荷服务器不实用，会造成内核空转

![img](.\img\v2-315644cfaca8ddb0c11565b1b4f2a643_720w.jpg)

![img](https://pics1.baidu.com/feed/63d9f2d3572c11df2f03e425f9f8f8d7f503c2c9.png?token=a799ec7f8d513d3b1c4d693436202af6)

### 轮询与中断

起初的纯轮询模式是指收发包完全不使用任何中断，集中所有运算资源用于报文处理。DPDK纯轮询模式 是指收发包完全不使用中断处理的高吞吐率的方 式。DPDK 所有的收发包有关的中断在物理端口初始化的时候都会关闭，也就是说，CPU 这边在任何时候都不会收到收包或者发包成功的中 断信号，也不需要任何收发包有关的中断处理。具体收发包流程参见之后的文章单独说明。网络应用中可能存在的潮汐效应，在某些时间段网络数据 流量可能很低，甚至完全没有需要处理的包，这样就会出现在高速端口 下低负荷运行的场景，而完全轮询的方式会让处理器一直全速运行，明显浪费处理能力和不节能。因此在 DPDK R2.1 和 R2.2 陆续添加了收包中断与轮询的混合模式的支持，类似 NAPI 的思路，用户可以根据实际应 用场景来选择完全轮询模式，或者混合中断轮询模式。而且，完全由用 户来制定中断和轮询的切换策略，比如什么时候开始进入中断休眠等待收包，中断唤醒后轮询多长时间，等等。

###多线程

多线程的初衷是提高整体应用程序的性能，但是如果不加注意，就会将多线程的创建和销毁开销，锁竞争，访存冲突，cache 失效，上下文切换等诸多消耗性能的因素引入进来。为了进一步提高性能，就必须仔细斟酌考虑线程在CPU不同核上的分布情况，这也就是常说的多核编程。多核编程和多线程有很大的不同：多线程是指每个 CPU 上可以运行多个线程，涉及到线程调度、锁机制以及上下文的切换；而多核则是每个 CPU 核一个线程，核心之间访问数据无需上锁。为了最大限度减少线程调度的资源消耗，需要将 Linux 绑定在特定的核上，释放其余核心来专供应用程序使用。DPDK 的线程基于 pthread 接口创建，属于抢占式线程模型，受内核 调度支配。DPDK 通过在多核设备上创建多个线程，每个线程绑定到单独的核上，减少线程调度的开销，以提高性能。DPDK 的线程可以作为控制线程，也可以作为数据线程。在 DPDK 的一些示例中，控制线程一般绑定到 MASTER 核上，接受用户配置，并 传递配置参数给数据线程等；数据线程分布在不同核上处理数据包。同时还需要考虑 CPU 特性和系统是否支持 NUMA 架构，如果支持的话，不同插槽上 CPU 的进程要避免访问远端内存，尽量访问本端内存。

### CPU亲核性

当处理器进入多核架构后，自然会面对一个问题，按照什么策略将 任务线程分配到各个处理器上执行。众所周知的是，这个分配工作一般 由操作系统完成。负载均衡当然是比较理想的策略，按需指定的方式也 是很自然的诉求，因为其具有确定性。简单地说，CPU 亲和性（Core affinity）就是一个特定的任务要在某 个给定的 CPU 上尽量长时间地运行而不被迁移到其他处理器上的倾向 性。这意味着线程可以不在处理器之间频繁迁移。这种状态正是我们所 希望的，因为线程迁移的频率小就意味着产生的负载小。将线程与CPU绑定，最直观的好处就是提高了 CPU Cache 的命中 率，从而减少内存访问损耗，提高程序的速度。在 Linux内核 中，所有的线程都有一个相关的数据结构，称为 task_struct。这个结构非常重要，原因有很多；其中与[亲和性](https://www.zhihu.com/search?q=亲和性&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2230499883})相关度最 高的是cpus_allowed 位掩码。这个位掩码由 n 位组成，与系统中的 n 个逻 辑处理器一一对应。具有 4 个物理 CPU 的系统可以有 4 位。如果这些 CPU 都启用了超线程，那么这个系统就有一个 8 位的位掩码。如果针对某个线程设置了指定的位，那么这个线程就可以在相关的 CPU上运行。因此，如果一个线程可以在任何 CPU 上运行，并且能够根 据需要在处理器之间进行迁移，那么位掩码就全是 1。实际上，在 Linux 中，这就是线程的默认状态。DPDK  通过把线程绑定到逻辑核的方法来避免跨核任务中的切换开 销，但对于绑定运行的当前逻辑核，仍然可能会有线程切换的发生，若希望进一步减少其他任务对于某个特定任务的影响，在亲和的基础上更 进一步，可以采取把逻辑核从内核调度系统剥离的方法。

### 大页表

内存的访问速度永远也赶不上cache和cpu的频率，为了能让性能平行扩展，最好是少访问。

减少访存次数来避免cache misses是我们设计的目标。

好处

- **避免使用swap**
  所有大页以及大页表都以共享内存存放在共享内存中，永远都不会因为内存不足而导致被交换到磁盘swap分区中。而linux系统默认的4K大小页面，是有可能被交换到swap分区的， 大页则永远不会。通过共享内存的方式，使得所有大页以及页表都存在内存，避免了被换出内存会造成很大的性能抖动
- **减少页表开销**
  由于所有进程都共享一个大页表，减少了页表的开销，无形中减少了内存空间的占用， 使系统支持更多的进程同时运行。
- **减轻TLB的压力**
  我们知道TLB是直接缓存虚拟地址与物理地址的映射关系，用于提升性能，省去查找page table减少开销，但是如果出现的大量的TLB miss，必然会给系统的性能带来较大的负面影响，尤其对于连续的读操作。使用hugepages能大量减少页表项的数量，也就意味着访问同样多的内容需要的页表项会更少，而通常TLB的槽位是有限的，一般只有512个，所以更少的页表项也就意味着更高的TLB的命中率
- **减轻查内存的压力**
  每一次对内存的访问实际上都是由两次抽象的内存操作组成。如果只要使用更大的页面，自然总页面个数就减少了，那么原本在页表访问的瓶颈也得以避免，页表项数量减少，那么使得很多页表的查询就不需要了。例如申请2M空间，如果4K页面，则一共需要查询512个页面，现在每个页为2M，只需要查询一个页就好了

默认下 Linux 采用 4KB 为一页，页越小内存越大，页表的开销越大，页表的内存占用也越大。CPU 有TLB（Translation Lookaside Buffer）成本高所以一般就只能存放几百到上千个页表项。如果进程要使用 64G 内存，则 64G/4KB=16000000（一千六百万）页，每页在页表项中占用 16000000 * 4B=62MB。如果用HugePage 采用 2MB 作为一页，只需 64G/2MB=2000，数量不在同个级别。而 DPDK 采用 HugePage，在 x86-64下 支持 2MB、1GB 的页大小，几何级的降低了页表项的大小，从而减少 TLB-Miss 。

### 无锁机制

实际上 DPDK 内部也有读写锁，LINUX 系统本身也支持无锁操作，并且 DPDK 内部的无锁机制实现原理同 LINUX 系统提供的无锁机制的实现原理类似。两者都采用无锁环形队列的方式，采用[环形队列](https://www.zhihu.com/search?q=环形队列&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2230499883})的好处是，当一个数据元素被用掉后，其余数据元素不需要移动其存储位置，从而减少拷贝，提高效率。LINUX 系统如果仅仅有一个读用户和一个写用户，那么不需要添加互斥保护机制就可以 保证数据的正确性。但是，如果有多个读写用户访问环形缓冲区，那么 必须添加互斥保护机制来确保多个用户互斥访问环形缓冲区。DPDK 的无锁环形队列无论是单用户读写还是多用户读写都不需要使用互斥锁保护。

### cache预取

处理器从一级 Cache 读取数据需要3~5个时 钟周期，二级是十几个时钟周期，三级是几十个时钟周期，而内存则需 要几百个时钟周期。DPDK 必须保证所有需要读取的数据都在 Cache 中，否则一旦出现 Cache 不命中，性能将会严重下降。为了保证这点，DPDK 采用 了多种技术来进行优化，预取只是其中的一种

### 利用UIO技术

为了让驱动运行在用户态，Linux 提供 UIO 机制。使用 UIO 可以通过read感知中断，通过 mmap 实现和网卡的通讯

#NUMA架构

NUMA 全称 Non-Uniform Memory Access，译为“**非一致性内存访问**”。这种构架下，不同的内存器件和CPU核心从属不同的 Node，每个 Node 都有自己的集成内存控制器（IMC，Integrated Memory Controller）。

在 Node 内部，架构类似SMP，使用 IMC Bus 进行不同核心间的通信；不同的 Node 间通过QPI（Quick Path Interconnect）进行通信

# 核心组件

##1. 环形缓冲区管理(librte_ring)

一个无锁的多生产者，多消费者的FIFO表处理接口，可用于不同核之间或是逻辑核上处理单元之间的通信。

##2. 内存池管理(librte_mempool)

主要职责是在内存中分配用来存储对象的 `pool`。 每个 `pool`以名称来唯一标识，并且使用一个`ring`来存储空闲的对象节点。 它还提供了一些其他的服务，如针对每个处理器核心的缓存或者一个能通过添加 padding 来使对象均匀分散在所有内存通道的对齐辅助工具。

## 3. 网络报文缓冲区管理(librte_mbuf)

它提供了创建、释放报文缓存的能力，DPDK 应用程序可能使用这些报文缓存来存储数据包。，这个缓存通常在程序开始时通过 DPDK 的 `mempool` 库创建。 这个库提供了创建和释放 mbuf 的 API，能用来暂存数据包。

## 4. 定时器管理(librte_timer)

这个模块为 DPDK 的执行单元提供了异步执行函数的能力，也能够周期性的触发函数。它是通过环境抽象层 EAL 提供的能力来获取的精准时间。

# 环境抽象层（EAL）

EAL 是用于为 DPDK 程序提供底层驱动能力抽象的，它使 DPDK 程序不需要关注下层具体的网卡或者操作系统，而只需要利用 EAL 提供的抽象接口即可，EAL 会负责将其转换为对应的 API。

EAL 主要提供了以下能力的抽象接口：

- DPDK的加载和启动：DPDK和指定的程序链接成一个独立的进程，并以某种方式加载
- CPU亲和性和分配处理：DPDK提供机制将执行单元绑定到特定的核上，就像创建一个执行程序一样。
- 系统内存分配：EAL实现了不同区域内存的分配，例如为设备接口提供了物理内存。
- PCI地址抽象：EAL提供了对PCI地址空间的访问接口
- 跟踪调试功能：日志信息，堆栈打印、异常挂起等等。
- 公用功能：提供了标准libc不提供的自旋锁、原子计数器等。
- CPU特征辨识：用于决定CPU运行时的一些特殊功能，决定当前CPU支持的特性，以便编译对应的二进制文件。
- 中断处理：提供接口用于向中断注册/解注册回掉函数。
- 告警功能：提供接口用于设置/取消指定时间环境下运行的毁掉函数。

在 Linux 中，DPDK 程序是以 pthread 创建的用户态程序形式存在的，会有一个初始线程来负责 EAL 的初始化，EAL 会使用 `mmap` 来操作 `hugetlbfs`，具体流程如下图所示。

![dpdk-architecture](\img\dpdk-architecture.png)

# 通用流`rte_flow`

rte_flow 提供了一种通用的方式来配置硬件以匹配特定的 Ingress 或 Egress 流量，根据用户的任何配置规则对其进行操作或查询相关计数器。

这种通用的方式细化后就是一系列的流规则，每条流规则由多种匹配模式和动作列表组成。

一个流规则可以具有几个不同的动作(如在将数据重定向到特定队列之前执行计数，封装，解封装等操作)， 而不是依靠几个规则来实现这些动作，应用程序操作具体的硬件实现细节来顺序执行。

```c
#define MAX_PATTERN_NUM        3
#define MAX_ACTION_NUM        2

struct rte_flow {
    struct rte_flow_attr attr;
    struct rte_flow_item pattern[MAX_PATTERN_NUM];
    struct rte_flow_action action[MAX_ACTION_NUM];
}
```

## 1. 属性`rte_flow_attr`

a. 组`group`

流规则可以通过为其分配一个公共的组号来分组，通过`jump`的流量将执行这一组的操作。较低的值具有较高的优先级。组0具有最高优先级。

b. 优先级`priority`

可以将优先级分配给流规则。像Group一样，较低的值表示较高的优先级，0为最大值。

组和优先级是任意的，取决于应用程序，它们不需要是连续的，也不需要从0开始，但是最大数量因设备而异，并且可能受到现有流规则的影响。

c. 流量方向`ingress or egress`

流量规则可以应用于入站和/或出站流量（Ingress/Egress）。

## 2. 模式条目`rte_flow_item`

模式条目类似于一套正则匹配规则，用来匹配目标数据包，其结构如代码所示。

```c
struct rte_flow_item {
    enum rte_flow_item_type type; /**< Item type. */
    const void *spec; /**< Pointer to item specification structure. */
    const void *last; /**< Defines an inclusive range (spec to last). */
    const void *mask; /**< Bit-mask applied to spec and last. */
};
```

首先模式条目`rte_flow_item_type`可以分成两类：

- 用于根据协议类型来匹配（如 ANY、ETH、VLAN、IPv4、UDP、TCP 等）
- 用于根据数据或影响模式来匹配（如 END、VOID、INVERT、PORT 等）

同时每个条目可以最多设置三个相同类型的结构：

- `spec`：精准匹配的数值
- `last`：与`spec` 组成一个范围
- `mask`：应用于`spec`和`last`的位掩码

a. ANY

可以匹配任何协议，还可以一个条目匹配多层协议。

```c
item.type = RTE_FLOW_ITEM_TYPE_ANY
struct rte_flow_item_any any = {.num = 2}
item.spec = &any /** 覆盖几层 */
item.last = &any /** 至多覆盖几层 */
```

b. ETH

```c
item.type = RTE_FLOW_ITEM_TYPE_ETH
struct rte_flow_item_eth eth = {
        .dst= dst_mac,
        .src= src_mac
}
item.spec = &eth
```

c. IPv4

```c
item.type = RTE_FLOW_ITEM_TYPE_IPV4
struct rte_flow_item_ipv4 ip = {
    .hdr = {
        .dst_addr = htonl(dest_ip),
        .src_addr = htonl(src_ip)
    }
}
item.spec = &ip
```

d. TCP

```c
item.type = RTE_FLOW_ITEM_TYPE_TCP
struct rte_tcp_hdr {
    rte_be16_t src_port; /**< TCP source port. */
    rte_be16_t dst_port; /**< TCP destination port. */
    rte_be32_t sent_seq; /**< TX data sequence number. */
    rte_be32_t recv_ack; /**< RX data acknowledgment sequence number. */
    uint8_t  data_off;   /**< Data offset. */
    uint8_t  tcp_flags;  /**< TCP flags */
    rte_be16_t rx_win;   /**< RX flow control window. */
    rte_be16_t cksum;    /**< TCP checksum. */
    rte_be16_t tcp_urp;  /**< TCP urgent pointer, if any. */
} tcp_hdr
struct rte_flow_item_tcp tcp = {
    .hdr = tcp_hdr
}
item.spec = &tcp
```

## 3. 操作`rte_flow_action`

操作用于对已经匹配到的数据包进行处理，同时多个操作也可以进行组合以实现一个流水线处理。

```c
struct rte_flow_action {
    enum rte_flow_action_type type; /**< Action type. */
    const void *conf; /**< Pointer to action configuration object. */
};
```

首先操作类别可以分成三类：

- 修改匹配流量去向的操作，例如删除或分配一个特定的目的地。
- 修改匹配流量内容或其属性的操作，包括添加/删除封装、加密、压缩和标记。
- 与流规则本身相关的操作，例如更新计数器或使其不终止（`PASSTHRU`进行复制）。

a. `MARK`

对流量进行标记，会设置`PKT_RX_FDIR`和`PKT_RX_FDIR_ID`两个 FLAG，具体的值可以通过`hash.fdir.hi`获得。

```c
// 下发规则
struct rte_flow_action_mark normal_mark = {.id = 0xbef};
struct rte_flow_action action = {
            .type = RTE_FLOW_ACTION_TYPE_MARK,
            .conf = &normal_mark,
};

// 提取数据包中的 MARK
if (ol_flags & PKT_RX_FDIR) {
        printf(" - FDIR matched ");
        if (ol_flags & PKT_RX_FDIR_ID)
            printf("ID=0x%x", m->hash.fdir.hi);
}
```

b. `QUEUE`

将流量上送到某个队列中

```c
// 下发规则
struct rte_flow_action_queue queue = { .index = 1 };
struct rte_flow_action action = {
            .type = RTE_FLOW_ACTION_TYPE_QUEUE,
            .conf = &normal_mark,
};

// 从队列中获取
for (i = 0; i < nr_queues; i++) {
      // 获取当前端口中缓存的数据包的地址
        nb_rx = rte_eth_rx_burst(port_id, i, mbufs, 32);
      if (nb_rx) {
                for (j = 0; j < nb_rx; j++) {
                      struct rte_mbuf *m = mbufs[j];
                      rte_pktmbuf_free(m);
                }
      }
}
```

c. `DROP`

将数据包丢弃

```c
RTE_FLOW_ACTION_TYPE_DROP
```

d. `COUNT`

对数据包进行计数，如果同一个`flow`里有多个`count`操作，则每个都需要指定一个独立的 id，`shared`标记的计数器可以用于统一端口的不同的`flow`一同进行计数。

```c
struct rte_flow_action_count shared_counter = {
    .shared = 1,
    .id = 100,
};

struct rte_flow_action_count dedicated_counter = {
    .shared = 0,
};

struct rte_flow_action action = {
    .type = RTE_FLOW_ACTION_TYPE_COUNT,
    .conf = &shared_counter,
};
```

e. `RAW_DECAP`

用来对匹配到的数据包进行拆包，一般用于隧道流量的剥离。在`action`定义的时候需要传入一个`data`用来指定匹配规则和需要移除的内容。

```c
// 定义匹配的数据包字段
struct rte_ether_hdr eth = {
    .ether_type = RTE_BE16(RTE_ETHER_TYPE_IPV4),
    .d_addr.addr_bytes = "\x01\x02\x03\x04\x05\x06",
    .s_addr.addr_bytes = "\x06\x05\x04\x03\x02\01" };
struct rte_flow_item_ipv4 ipv4 = {
    .hdr = {
        .next_proto_id = IPPROTO_UDP }};
struct rte_flow_item_udp udp = {
    .hdr = {
            .dst_port = rte_cpu_to_be_16(2152) }};
                /* Match on UDP dest port 2152 (GTP-U) */
struct rte_flow_item_gtp gtp;

// 将内容填充进去
size_t decap_size = sizeof(eth) + sizeof(ipv4) + sizeof(udp) +
            sizeof(gtp);
uint8_t decap_buf[decap_size];
uint8_t *bptr;
bptr = decap_buf;
memcpy(bptr, &eth, sizeof(eth));
bptr += sizeof(eth);
memcpy(bptr, &ipv4, sizeof(ipv4));
bptr += sizeof(ipv4);
memcpy(bptr, &udp, sizeof(udp));
bptr += sizeof(udp);
memcpy(bptr, &gtp, sizeof(gtp));
bptr += sizeof(gtp);

struct rte_flow_action_raw_decap decap = {
    .size = decap_size ,
    .data = decap_buf
};
```

f. `RSS`

对流量进行负载均衡的操作，他将根据提供的数据包进行哈希操作，并将其移动到对应的队列中。

```c
struct rte_flow_action_rss {
    enum rte_eth_hash_function func;    
    uint32_t level; 
    uint64_t types; /**< Specific RSS hash types (see ETH_RSS_*). */
    uint32_t key_len; /**< Hash key length in bytes. */
    uint32_t queue_num; /**< Number of entries in @p queue. */
    const uint8_t *key; /**< Hash key. */
    const uint16_t *queue; /**< Queue indices to use. */
}
```

其中的`level`属性用来指定使用第几层协议进行哈希：

- 0：默认操作，其代表自动判断使用哪一层进行操作
- 1：代表使用最外层
- 2-*：代表使用指定封装层进行

#常用API

## 1. 程序初始化

```c
// 获取有效网卡的个数
nb_ports = rte_eth_dev_count_avail();
if (nb_ports < 2 || (nb_ports & 1))
    rte_exit(EXIT_FAILURE, "Error: number of ports must be even\n");

/*
 * 初始化内存池
 *
 * @param name
 *   The name of the mbuf pool.
 * @param n
 *   The number of elements in the mbuf pool. The optimum size (in terms
 *   of memory usage) for a mempool is when n is a power of two minus one:
 *   n = (2^q - 1).
 * @param cache_size
 *   Size of the per-core object cache. See rte_mempool_create() for
 *   details.
 * @param priv_size
 *   Size of application private are between the rte_mbuf structure
 *   and the data buffer. This value must be aligned to RTE_MBUF_PRIV_ALIGN.
 * @param data_room_size
 *   Size of data buffer in each mbuf, including RTE_PKTMBUF_HEADROOM.
 * @param socket_id
 *   The socket identifier where the memory should be allocated. The
 *   value can be *SOCKET_ID_ANY* if there is no NUMA constraint for the
 *   reserved zone.
 */
mbuf_pool = rte_pktmbuf_pool_create("mbuf_pool", 40960, 128, 0,
                        RTE_MBUF_DEFAULT_BUF_SIZE,
                        rte_socket_id());
    if (mbuf_pool == NULL)
        rte_exit(EXIT_FAILURE, "Cannot init mbuf pool\n");

// 分配任务到每个核心上
static int lcore_hello(__rte_unused void *arg){
    unsigned lcore_id;
    lcore_id = rte_lcore_id();
    printf("hello from core %u\n", lcore_id);
    return 0;
}
rte_eal_remote_launch(lcore_hello, NULL, lcore_id);
```

## 2. 端口初始化

```c
struct rte_eth_conf port_conf = {
  .rxmode = {
      .split_hdr_size = 0,
  },
  .txmode = {
      .offloads =
          DEV_TX_OFFLOAD_VLAN_INSERT |
          DEV_TX_OFFLOAD_IPV4_CKSUM |
          DEV_TX_OFFLOAD_UDP_CKSUM |
          DEV_TX_OFFLOAD_TCP_CKSUM |
          DEV_TX_OFFLOAD_SCTP_CKSUM |
          DEV_TX_OFFLOAD_TCP_TSO,
  },
};

struct rte_eth_txconf txq_conf;
struct rte_eth_rxconf rxq_conf;
struct rte_eth_dev_info dev_info;

ret = rte_eth_dev_info_get(port_id, &dev_info);
if (ret != 0)
    rte_exit(EXIT_FAILURE,
         "Error during getting device (port %u) info: %s\n",
         port_id, strerror(-ret));

port_conf.txmode.offloads &= dev_info.tx_offload_capa;
printf(":: initializing port: %d\n", port_id);

ret = rte_eth_dev_configure(port_id,
              nr_queues, nr_queues, &port_conf);

if (ret < 0) {
    rte_exit(EXIT_FAILURE,
         ":: cannot configure device: err=%d, port=%u\n",
         ret, port_id);
}

rxq_conf = dev_info.default_rxconf;
rxq_conf.offloads = port_conf.rxmode.offloads;

txq_conf = dev_info.default_txconf;
txq_conf.offloads = port_conf.txmode.offloads;
```

## 3. 队列初始化

```c
for (i = 0; i < nr_std_queues; i++) {
        ret = rte_eth_rx_queue_setup(port_id, i, 512,
                     rte_eth_dev_socket_id(port_id),
                     &rxq_conf,
                     mbuf_pool);
        if (ret < 0) {
            rte_exit(EXIT_FAILURE,
                    ":: Rx queue setup failed: err=%d, port=%u\n",
                    ret, port_id);
        }
}

for (i = 0; i < nr_std_queues; i++) {
    ret = rte_eth_tx_queue_setup(port_id, i, 512,
            rte_eth_dev_socket_id(port_id),
            &txq_conf);
    if (ret < 0) {
        rte_exit(EXIT_FAILURE,
            ":: Tx queue setup failed: err=%d, port=%u\n",
            ret, port_id);
    }
}
```

# 进阶

##哈希库rte_hash

DPDK 提供了一个标准的哈希表的实现，能用来根据键值快速进行索引。

a. API

```c
// 初始化
struct rte_hash_parameters flow_hash_table_parameter = {
            .name = table_name, // 需保证唯一
            .entries = MAX_HASH_ENTRIES, 
            .key_len = sizeof(union ipv4_5tuple_host),
            .hash_func = ipv4_hash_crc, // 可以自行编写，可以使用自带的rte_hash_crc等
            .hash_func_init_val = 0,
    };
flow_hash_table = rte_hash_create(&flow_hash_table_parameter);

// 增加
// 对于重复的键值会直接进行覆盖，同时data的内存需要自行管理。
int rte_hash_add_key_data(const struct rte_hash *h, const void *key, void *data);

// 查找
// 对于存在的键值会返回一个非负的数，参数非法会返回-EINVAL，不存在则会返回-ENOENT。
int rte_hash_lookup_data(const struct rte_hash *h, const void *key, void **data);

// 删除
// 从哈希表中移除一个键值对，之后还需调用rte_hash_free_key_with_position清理键内存，以及使用rte_free释放值存储的内存。
int32_t rte_hash_del_key(const struct rte_hash *h, const void *key);

// 清理键内存
// 由于rte_hash会将键的内容复制一份进去，所以删除后需要对其存储该键的内存进行释放
int rte_hash_get_key_with_position(const struct rte_hash *h, const int32_t position,
			       void **key);

// 销毁
const void *key = 0;
void *data = 0;
uint32_t *next = 0;
uint32_t current = rte_hash_iterate(flow_hash_table, &key, &data, next);
for (; current != -ENOENT; current = rte_hash_iterate(flow_hash_table, &key, &data, next)) {
		int32_t del_key = rte_hash_del_key(flow_hash_table, key);
		rte_hash_free_key_with_position(flow_hash_table, del_key);
		rte_free(data);
}
rte_hash_free(flow_hash_table);
```

b. 自定义哈希

```c
#if defined(RTE_ARCH_X86) || defined(__ARM_FEATURE_CRC32)
#define EM_HASH_CRC 1
#endif

#ifdef EM_HASH_CRC
#include <rte_hash_crc.h>
#define DEFAULT_HASH_FUNC       rte_hash_crc
#else
#include <rte_jhash.h>
#define DEFAULT_HASH_FUNC       rte_jhash
#endif

static inline uint32_t ipv4_hash_crc(const void *data, __rte_unused uint32_t data_len,
              uint32_t init_val) {
    const union ipv4_5tuple_host *k;
    uint32_t t;
    const uint32_t *p;

    k = data;
    t = k->proto;
    p = (const uint32_t *) &k->port_src;

#ifdef EM_HASH_CRC
    init_val = rte_hash_crc_4byte(t, init_val);
    init_val = rte_hash_crc_4byte(k->ip_src, init_val);
    init_val = rte_hash_crc_4byte(k->ip_dst, init_val);
    init_val = rte_hash_crc_4byte(*p, init_val);
#else
    init_val = rte_jhash_1word(t, init_val);
    init_val = rte_jhash_1word(k->ip_src, init_val);
    init_val = rte_jhash_1word(k->ip_dst, init_val);
    init_val = rte_jhash_1word(*p, init_val);
#endif

    return init_val;
}
```

c. 快速获取五元组

```c
union ipv4_5tuple_host key = {.xmm=0};
mask0 = (rte_xmm_t) {.u32 = {BIT_8_TO_15, ALL_32_BITS,
                              ALL_32_BITS, ALL_32_BITS}};

// x86-64 且支持 SSE
static __rte_always_inline void 
get_ipv4_5tuple(struct rte_mbuf *m0, __m128i mask0, union ipv4_5tuple_host *key) {
    __m128i tmpdata0 = _mm_loadu_si128(
            rte_pktmbuf_mtod_offset(m0, __m128i *,
                                    sizeof(struct rte_ether_hdr) +
                                    offsetof(struct rte_ipv4_hdr, time_to_live)));

    key->xmm = _mm_and_si128(tmpdata0, mask0);
}

// ARM
static inline void
get_ipv4_5tuple(struct rte_mbuf *m0, int32x4_t mask0,
		union ipv4_5tuple_host *key)
{
	int32x4_t tmpdata0 = vld1q_s32(rte_pktmbuf_mtod_offset(m0, int32_t *,
				sizeof(struct rte_ether_hdr) +
				offsetof(struct rte_ipv4_hdr, time_to_live)));

	key->xmm = vandq_s32(tmpdata0, mask0);
}
```

##高性能获取时间

可以通过获取处理器寄存器中的周期数来高性能的获取精准时间，但要保证处理器运行在一个稳定的频率上，这样时间才是准确的。

```c
// 获取处理器自从开机以来的周期数
uint64_t rte_rdtsc()

// 获取处理器一秒的周期数
rte_get_tsc_hz()

time = rte_rdtsc() / rte_get_tsc_hz()
```

##能耗管理库rte_power

由于获取时间需要稳定处理器频率，所以可以利用 DPDK 提供的 API 对处理器进行配置，稳定的频率也有利于保证业务的稳定，能去除尖刺。

```c
// DPDK对处理器能耗进行接管
rte_power_init(lcore_id);

// 获取处理器可以运行的频率
uint32_t freqs[30];
rte_power_freqs(lcore_id, freqs, 30);
for (int i = 0; i < 30; ++i) {
		printf("\\t%u", freqs[i]);
}

// 设置处理器运行在哪个频率上
ret = rte_power_set_freq(lcore_id, 2);

// 停止接管
rte_power_exit(lcore_id);
```

## 配合cmake使用

```bash
# import dpdk library
find_package(PkgConfig REQUIRED)
pkg_search_module(LIBDPDK REQUIRED libdpdk)

# import thread
find_package(Threads REQUIRED)

link_directories(${LIBDPDK_LIBRARY_DIRS})
include_directories(${LIBDPDK_INCLUDE_DIRS})

target_link_libraries(smart_offload ${LIBDPDK_LIBRARIES} Threads::Threads)
```