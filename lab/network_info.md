# [组播MAC地址和各类IP地址](https://www.cnblogs.com/lifan3a/articles/6650936.html)

MAC地址是以太网二层使用的一个48bit（6字节十六进制数）的地址，用来标识设备位置。MAC地址分成两部分，前24位是组织唯一标识符（OUI, Organizationally unique identifier），后24位由厂商自行分配。

  MAC地址有单播、组播、广播之分。单播地址(unicast address)表示单一设备、节点，多播地址或者组播地址(multicast address、group address)表示一组设备、节点，广播地址(broadcast address)是组播的特例，表示所有地址，用全F表示：FF-FF-FF-FF-FF-FF。当然，三层的IP地址也有单播、组播、广播之分。

  48bit的MAC地址一般用6字节的十六进制来表示，如XX-XX-XX-XX-XX。IEEE 802.3规定：以太网的第48bit(2012-04-11修改为The first bit) 用于表示这个地址是组播地址还是单播地址。如果这一位是0，表示此MAC地址是单播地址，如果这位是1，表示此MAC地址是多播地址。见IEEE 802.3 3.2.3 Address fields：“The first bit (LSB) shall be used in the Destination Address field as an address type designation bit to identify the Destination Address either as an individual or as a group address. If this bit is 0, it shall indicate that the address field contains an individual address. If this bit is 1, it shall indicate that the address field contains a group address that identifies none, one or more, or all of the stations connected to the LAN. In the Source Address field, the first bit is reserved and set to 0.”

  以太网线路上按“Big Endian”字节序传送报文（也就是最高字节先传送，关于字节序请参考相关文档），而比特序是”Little Endian”（也就是最低位先传送）。一个十六进制表示法表示的MAC地址01-80-C2-00-00-00，传送时的bit顺序就是：1000 0000 0000 0001 0100 0011 0000 0000 0000 0000 0000 0000

  注意图上的第47bit(2012-04-11修改为The second bit)，这一位表示MAC地址是全球唯一地址还是本地地址，0表示全球唯一地址，1表示本地唯一地址。这一位也叫G/L位。

  对于网络设备上固化的MAC地址，因为它唯一标识这个设备，所以只能是单播地址，也就是MAC帧里面的Source地址第48位(2012-04-11修改为The first bit)只能为0。   

  我们常说有2的48次方个MAC地址可供网络设备使用，这些地址可以多到给地球上每一粒沙子分配一个地址，其实这个数量要打折扣的，因为MAC地址虽然有这么多，但真正用在网卡上并且全球唯一的只有2的46次方个：第48bit一定是0，第47bit一定是0。

  这也就引出了一个有意思的现象：随便找一台PC，观察一下它的网卡地址，第1字节的十六进制数一般是4的倍数；查看一下IEEE分配的OUI(http://standards.ieee.org/develop/regauth/oui/oui.txt)，第1字节的十六进制数也一般是4的倍数（早期以太网没有本地地址的概念，所以分配的OUI里面G/L bit也可能是1），这种情况下就不是4的倍数了，但肯定是2的倍数，因为第48位只能是0。

  关于组播地址，有这么个误解：MAC地址第1字节必须是0x01才表示组播地址，连TCP/IP详解上也这么说（见中文版12.4.2第一段）。IEEE 802.3里面已经明确说明了只要第48bit是1就表示组播地址，所以无论MAC地址第1字节是0x01、0xC1或者是0x33都表示这个MAC地址 是组播地址（以0x33开头的表示IPV6对应的二层组播地址）。之所以有这样的误解，是因为到目前为止，大部分组播MAC地址的第1字节都是0x01。 如：

01-80-C2-00-00-00(STP协议使用)

01-80-C2-00-00-01(MAC Control的PAUSE帧使用)

01-80-C2-00-00-02(Slow Protocol: 802.3ah OAM/ LACP 协议都用这个地址，这个地址很有故事，有多少软件处理这个地址会出问题啊！)

01-00-5E-xx-xx-xx(IP组播地址对应的二层组播地址)。

完整的列表见http://standards.ieee.org/develop/regauth/grpmac/public.html

  之所以大部分组播地址都以01-80-C2和01-00-5E开头，那是因为使用这些组播地址的协议都是带头大哥IEEE和IANA名下的，它们的OUI 分别是00-80-C2和00-00-5E是，变成组播地址就是01-80-C2和01-00-5E了，当然，除了带头大哥霸占的这些组播地址，还有 01-00-0C-CC-CC-CC这样的地址，这个地址是Cisco霸占的，Cisco的OUI是00-00-0C。





#网络分层

http://blog.iis7.com/article/24224.html  二层转发的介绍文章

https://www.cnblogs.com/patrick-yeh/p/14189219.html  ↓

**二层网络结构模型**： 核心层和接入层（没有汇聚层）
**三层网络结构模型**： 核心层、汇聚层和接入层

**二层网络**的组网能力非常有限，所以一般只是**用来搭建小局域网**，
二层网络结构模式运行简便交换机根据MAC地址表进行数据包的转发，有则转发，无则泛洪，即将数据包广播发送到所有端口，如果目的终端收到给出回应，那么交换机就可以将该MAC地址添加到地址表中，这是交换机对MAC地址进行建立的过程，但这样频繁的对未知的MAC目标的数据包进行广播，在大规模的网络架构中形成的网络风暴是非常庞大的，这也很大程度上限制了二层网络规模的扩大。

**三层网络结构可以组建大型的网络。**

（1）**核心层**是整个网络的支撑脊梁和数据传输通道，必须配备高性能的数据冗余转接设备和防止负载过剩的均衡负载的设备，以降低各核心层交换机所需承载的数据量。（网络的高速交换主干）

（2）**汇聚层**是连接网络的核心层和各个接入的应用层，在两层之间承担“媒介传输”的作用。
汇聚层应该具备以下功能：
1，实施安全功能（划分VLAN和配置ACL）
2，工作组整体接入功能
3，虚拟网络过滤功能。
因此，**汇聚层设备应采用三层交换机**。
（提供基于策略的连接）

（3）**接入层**的面向对象主要是终端客户，为终端客户提供接入功能。（将工作站接入网络）

二层网络 仅仅通过MAC寻址即可实现通讯，但仅仅是同一个冲突域内；
三层网络则需要通过IP路由实现跨网段的通讯，可以跨多个冲突域。

三层交换机在**一定程度**上可以替代路由器，但是应该清醒的认识到三层交换机出现最重要的目的是加快大型局域网内部的数据交换，所具备的路由功能也多是围绕这一目的而展开的，所以他的路由功能没有同一档次的专业路由器强，在安全、协议支持等方面还有许多欠缺，并**不能完全取**代路由器工作。

**在实际应用过程中，典型的做法是**： 处于同一个局域网中的各个子网的互联以及局域网中VLAN间的路由，用三层交换机来代替路由器。
而只有局域网与公网互联之间要实现跨地域的网络访问时，才通过专业路由器。

二层网络结构：**对于公司部门容易物理位置易变动的企业。**
三层网络结构：**新型大楼部署局域网，物理位置变动小，频率比较小的公司。**

二层网络：以**部门**为单位划分。
三层网络：以**楼层配线间**为单位划分。