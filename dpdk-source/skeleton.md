main函数：

1. **初始化 EAL**
2. **以太网端口数需要为偶数（在这个程序中一个口接收，往另一个口转发）**
3. **分配mbuf `rte_pktmbuf_pool_create()`**
4. **初始化所有端口 转入 `port_init(portid, mbuf_pool)`**
5. **调用主线程开始basicfwd。**

```c
int
main(int argc, char *argv[])
{
	struct rte_mempool *mbuf_pool; // 指向内存池结构的指针
	unsigned nb_ports; // 网口个数
	uint16_t portid;   // 网口号
 
	/* Initialize the Environment Abstraction Layer (EAL). */
	// 初始化 EAL 
	int ret = rte_eal_init(argc, argv);
	if (ret < 0)
		rte_exit(EXIT_FAILURE, "Error with EAL initialization\n");
 
        //当解析完了EAL的参数之后，argc减去EAL参数的个数同时argv后移这么多位，
	//这样就能保证后面解析程序参数的时候跳过了前面的EAL参数。	
	argc -= ret;   
	argv += ret;
 
	/* Check that there is an even number of ports to send/receive on. */
	//以太网端口数需要为偶数
	nb_ports = rte_eth_dev_count_avail(); // 获取当前可用以太网设备的总数
	if (nb_ports < 2 || (nb_ports & 1))   // 检查端口个数是否小于两个或者是奇数，则出错。
		rte_exit(EXIT_FAILURE, "Error: number of ports must be even\n");
 
	/* Creates a new mempool in memory to hold the mbufs. */
	mbuf_pool = rte_pktmbuf_pool_create("MBUF_POOL", NUM_MBUFS * nb_ports,
		MBUF_CACHE_SIZE, 0, RTE_MBUF_DEFAULT_BUF_SIZE, rte_socket_id());  //rte_socket_id()返回正在运行的lcore所对应的物理socket。socket的文档在 lcore中
 
	if (mbuf_pool == NULL)
		rte_exit(EXIT_FAILURE, "Cannot create mbuf pool\n");
 
	/* Initialize all ports. 在每个网口上初始化*/
	RTE_ETH_FOREACH_DEV(portid)
		if (port_init(portid, mbuf_pool) != 0)
			rte_exit(EXIT_FAILURE, "Cannot init port %"PRIu16 "\n",
					portid);
 
	if (rte_lcore_count() > 1) // basicfwd只需要使用一个逻辑核
		printf("\nWARNING: Too many lcores enabled. Only 1 used.\n");
 
	/* Call lcore_main on the master core only. */
	// 仅仅一个主线程调用
        // 这个程序纯粹地把一个网口收到的包从另一个网口转发出去，就像是一个repeater，中间没有其他任何处理。
	lcore_main();
 
	return 0;
 
 
	
    // dpdk用mbuf保存packet，mempool用于操作mbuf
    /* rte_pktmbuf_pool_create() 创建并初始化mbuf池，是 rte_mempool_create 这个函数的封装。
        五个参数：
        1. mbuf的名字 "MBUF_POOL"
        2. mbuf中的元素个数。每个端口给了8191个
        3. 每个核心的缓存大小，如果该参数为0 则可以禁用缓存。本程序中是250
        4. 每个mbuf中的数据缓冲区大小
        5. 应分配内存的套接字标识符。
        返回值：分配成功时返回指向新分配的mempool的指针。
        mempool的指针会传给 port_init 函数，用于 setup rx queue
    */
}
```

端口初始化`port_init(portid, mbuf_pool)`：

1. 获取可用 eth 的个数
2. 配置网卡设备
3. 每个 port 1 个 rx 队列
4. 每个 port 1 个 tx 队列
5. 启用网卡设备
6. 设置网卡混杂模式

```c
static inline int
port_init(uint16_t port, struct rte_mempool *mbuf_pool)
{
	struct rte_eth_conf port_conf = port_conf_default; // 这个rte_eth_conf结构体在配置网卡时要用到
	const uint16_t rx_rings = 1, tx_rings = 1;         // 每个网口有多少rx和tx队列，这里都为1
	uint16_t nb_rxd = RX_RING_SIZE; // 接收环大小
	uint16_t nb_txd = TX_RING_SIZE; // 发送环大小
	int retval;
	uint16_t q;
	struct rte_eth_dev_info dev_info; // 用于获取以太网设备的信息，setup queue 时用到
	struct rte_eth_txconf txconf;     // setup tx queue 时用到
 
	if (!rte_eth_dev_is_valid_port(port)) // 检查设备的port_id是否已连接
		return -1;
 
	rte_eth_dev_info_get(port, &dev_info); //查询以太网设备的信息，参数port指示以太网设备的网口标识符，第二个参数指向要填充信息的类型rte_eth_dev_info的结构的指针
	if (dev_info.tx_offload_capa & DEV_TX_OFFLOAD_MBUF_FAST_FREE)
		port_conf.txmode.offloads |=
			DEV_TX_OFFLOAD_MBUF_FAST_FREE;
 
   
 
	/* Configure the Ethernet device. 配置网卡*/
 
	// rte_eth_dev_configure() 
    /* 四个参数
        1. port id
        2. 要给该网卡配置多少个收包队列 这里是一个
        3. 要给该网卡配置多少个发包队列 也是一个
        4. 结构体指针类型 rte_eth_conf *
    */
	retval = rte_eth_dev_configure(port, rx_rings, tx_rings, &port_conf);
	if (retval != 0)
		return retval;
 
    //检查Rx和Tx描述符的数量是否满足以太网设备信息中的描述符限制，否则将它们调整为边界。
	retval = rte_eth_dev_adjust_nb_rx_tx_desc(port, &nb_rxd, &nb_txd); 
	if (retval != 0)
		return retval;
 
	/* Allocate and set up 1 RX queue per Ethernet port. */
	/*  rte_eth_rx_queue_setup()
    配置rx队列需要六个参数
        1. port id
        2. 接收队列的索引。要在[0, rx_queue - 1] 的范围内（先前rte_eth_dev_configure中配置的）
        3. 为接收环分配的接收描述符数。（环的大小）
        4. socket id。 如果是 NUMA 架构 就使用 rte_eth_dev_socket_id(port)获取port所对应的以太网设备所连接上的socket的id；若不是NUMA，该值可以是宏SOCKET_ID_ANY
        5. 指向rx queue的配置数据的指针。如果是NULL，则使用默认配置。
        6. 指向内存池mempool的指针，从中分配mbuf去操作队列。
    */ 
	for (q = 0; q < rx_rings; q++) {
		retval = rte_eth_rx_queue_setup(port, q, nb_rxd,
				rte_eth_dev_socket_id(port), NULL, mbuf_pool);
		if (retval < 0)
			return retval;
	}
 
	txconf = dev_info.default_txconf;
	txconf.offloads = port_conf.txmode.offloads;
	/* Allocate and set up 1 TX queue per Ethernet port. */
	/* rte_eth_tx_queue_setup()
    配置tx队列需要五个参数（不需要mempool）
        1. port id
        2. 发送队列的索引。要在[0, tx_queue - 1] 的范围内（先前rte_eth_dev_configure中配置的）
        3. 为发送环分配的接收描述符数。（自定义环的大小）
        4. socket id
        5. 指向tx queue的配置数据的指针，结构体是rte_eth_txconf。
    */
 
	for (q = 0; q < tx_rings; q++) {
		retval = rte_eth_tx_queue_setup(port, q, nb_txd,
				rte_eth_dev_socket_id(port), &txconf);
		if (retval < 0)
			return retval;
	}
 
	/* Start the Ethernet port.  启动设备*/
	retval = rte_eth_dev_start(port);
	if (retval < 0)
		return retval;
 
	/* Display the port MAC address. */
	struct ether_addr addr;
	rte_eth_macaddr_get(port, &addr);
	printf("Port %u MAC: %02" PRIx8 " %02" PRIx8 " %02" PRIx8
			   " %02" PRIx8 " %02" PRIx8 " %02" PRIx8 "\n",
			port,
			addr.addr_bytes[0], addr.addr_bytes[1],
			addr.addr_bytes[2], addr.addr_bytes[3],
			addr.addr_bytes[4], addr.addr_bytes[5]);
 
	/* Enable RX in promiscuous mode for the Ethernet device. */
	rte_eth_promiscuous_enable(port); //设置网卡为混杂模式    // 指一台机器能够接收所有经过它的数据流，而不论其目的地址是否是他。
 
	return 0;
}
```

在单核上调用`lcore_main()`:

1. 检查收发网卡是否在同一 NUMA 节点
2. 死循环收发包，Ctrl+C 退出 :

- 从网卡读包
- 发送到网卡

```c
static __attribute__((noreturn)) void
lcore_main(void)
{
	uint16_t port;
 
	/*
	 * Check that the port is on the same NUMA node as the polling thread
	 * for best performance.
	 */
	// 当有NUMA结构时，检查网口是否在同一个NUMA node节点上，只有在一个NUMA node上时线程轮询的效率最好
	RTE_ETH_FOREACH_DEV(port)
		if (rte_eth_dev_socket_id(port) > 0 &&
				rte_eth_dev_socket_id(port) !=
						(int)rte_socket_id())
			printf("WARNING, port %u is on remote NUMA node to "
					"polling thread.\n\tPerformance will "
					"not be optimal.\n", port);
 
	printf("\nCore %u forwarding packets. [Ctrl+C to quit]\n",
			rte_lcore_id());
 
	/* Run until the application is quit or killed. */
	for (;;) {
		/*
		 * Receive packets on a port and forward them on the paired
		 * port. The mapping is 0 -> 1, 1 -> 0, 2 -> 3, 3 -> 2, etc.
		 */
		/*
            一个端口收到包，就立刻转发到另一个端口
            0 和 1
            2 和 3
            ……
        */
		RTE_ETH_FOREACH_DEV(port) {
 
			/* Get burst of RX packets, from first port of pair. */
			struct rte_mbuf *bufs[BURST_SIZE];  // mbuf的结构体 收到的包存在这里，也是要发出去的包
			const uint16_t nb_rx = rte_eth_rx_burst(port, 0,
					bufs, BURST_SIZE);
 
			if (unlikely(nb_rx == 0)) // 返回值是实际收到的数据包数
				continue;
 
			/* Send burst of TX packets, to second port of pair. */
			const uint16_t nb_tx = rte_eth_tx_burst(port ^ 1, 0, 
					bufs, nb_rx);// port 异或 1 --> 0就和1是一对，2就和3是一对。
                    // 0 收到包就从 1 转发， 3 收到包 就从 2 口转发。
 
 
			/* Free any unsent packets. */   //手动释放没有发送出去的 mbuf  // 用unlikely宏代表这种情况不太可能出现？
			if (unlikely(nb_tx < nb_rx)) {
				uint16_t buf;
				for (buf = nb_tx; buf < nb_rx; buf++)
					rte_pktmbuf_free(bufs[buf]);
			}
		}
 
	}
}
```

```c
    	/* 收包函数：rte_eth_rx_burst 从以太网设备的接收队列中检索一连串（burst收发包机制）输入数据包。检索到的数据包存储在rte_mbuf结构中。
	参数四个
		1. port id （收到哪个网口）
		2. 队列索引 （的哪一条队列），范围要在[0, rx_queue - 1] 的范围内（rte_eth_dev_configure中的）
		3. 指向 rte_mbuf 结构的 指针数组 的地址。要够容纳第四个参数所表示的数目的指针。（把收到的包存在哪里？）
		4. 要检索的最大数据包数
	rte_eth_rx_burst()是一个循环函数，从RX队列中收包达到设定的最大数量为止。
	收包操作：
	1. 根据NIC的RX描述符信息，初始化rte_mbuf数据结构。
	2. 将rte_mbuf（也就是数据包）存储到第三个参数所指示的数组的下一个条目。
	3. 从mempool分配新的的rte_mbuf
	*/
 
	/* Send burst of TX packets, to second port of pair. */
	/* 发包函数：rte_eth_tx_burst 在由port id指示的以太网设备的传输队列（由索引指示）发送一连串输出数据包。
	参数四个：
		1. port id（从哪个网口）
		2. 队列索引（的哪条队列发出），范围要在[0, tx_queue - 1] 的范围内（rte_eth_dev_configure中的）
		3. 指向包含要发送的数据包的 rte_mbuf 结构的 指针数组 的地址。（要发送的包的内容在哪里）
		4. 要发送的数据包的最大数量。
	返回值是发送的包的数量。
	发包操作：
	1. 选择发包队列中下一个可用的描述符
	2. 使用该描述符发送包，之后释放对应的mempool空间
	3. 再根据 *rte_mbuf 初始化发送描述符
	*/
```
