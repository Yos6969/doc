# server.h

所有的服务器配置和一般的所有共享状态都定义在一个名为`server`, 类型的全局结构中`struct redisServer`。此结构中的几个重要字段是：

- `server.db`是存储数据的 Redis 数据库数组。
- `server.commands`是命令表。
- `server.clients`是连接到服务器的客户端的链表。
- `server.master`如果实例是副本，则为特殊客户端，即主服务器。

另一个重要的 Redis 数据结构是定义客户端的数据结构。过去叫它`redisClient`，现在叫`client`。该结构有很多字段，这里我们只展示主要的：

```c
struct client {
    int fd;
    sds querybuf;
    int argc;
    robj **argv;
    redisDb *db;
    int flags;
    list *reply;
    // ... many other fields ...
    char buf[PROTO_REPLY_CHUNK_BYTES];
}
```

客户端结构定义了一个*连接的客户端*：

- 该`fd`字段是客户端套接字文件描述符。
- `argc`并`argv`填充了客户端正在执行的命令，以便实现给定 Redis 命令的函数可以读取参数。
- `querybuf`累积来自客户端的请求，由 Redis 服务器根据 Redis 协议进行解析，并通过调用客户端正在执行的命令的实现来执行。
- `reply`并且`buf`是动态和静态缓冲区，用于累积服务器发送给客户端的回复。一旦文件描述符可写，这些缓冲区就会增量写入套接字。

正如您在上面的客户端结构中所见，命令中的参数被描述为`robj`结构。以下是完整的`robj` 结构，它定义了一个*Redis 对象*：

```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* lru time (relative to server.lruclock) */
    int refcount;
    void *ptr;
} robj;
```

# server.c

`main()`这是定义函数的 Redis 服务器的入口点。以下是启动 Redis 服务器的最重要步骤。

- `initServerConfig()`设置`server`结构的默认值。
- `initServer()`分配操作、设置监听套接字等所需的数据结构。
- `aeMain()`启动监听新连接的事件循环。

事件循环会定期调用两个特殊函数：

1. `serverCron()`定期调用（根据`server.hz`频率），并执行必须不时执行的任务，例如检查超时的客户端。
2. `beforeSleep()`每次触发事件循环时都会调用，Redis 服务一些请求，然后返回到事件循环中。

在 server.c 中，您可以找到处理 Redis 服务器其他重要事项的代码：

- `call()`用于在给定客户端的上下文中调用给定命令。
- `activeExpireCycle()``EXPIRE`处理通过命令设置生存时间的键的驱逐。
- `performEvictions()`当应该执行新的写入命令但根据`maxmemory`指令 Redis 内存不足时调用。
- 全局变量`redisCommandTable`定义所有 Redis 命令，指定命令的名称、实现命令的函数、所需参数的数量以及每个命令的其他属性。



# Redis持久化

## RDB（Redis DataBase）

在指定的**时间间隔**内将内存中的数据集**快照**写入磁盘，恢复时直接将快照文件读到内存里,也可以根据操作次数进行主动写入，比如在配置文件中设置 eg 20 3，20s内三次操作触发

- fork()创建子进程进行持久化
- 将数据写入到一个临时文件中
- 临时文件替换上次持久化好的文件

RDB的缺点是最后一次持久化后的数据可能丢失（因为存储快照有时间间隔）

优势：

- 适合大规模数据恢复
- 节省磁盘空间
- 恢复速度快

## AOF（Append Only File）

以日志的形式记录每个写操作，增量保存。只许追加文件但不可以改写文件

```
默认不开启，使用RDB
#可以同时开启RDB和AOF,当redis重启的时候优先加载AOF文件来恢复原始数据,因为在通常情况下AOF文件保存的数据要比RDB文件保存的数据集要完整。RDB的数据不实时,同时使用两者时服务器重启也只会找AOF文件。那要不要只使用AOF呢?建议不要,因为RDB更适合备分数据库(AOF在不断变换不好备分)快速重启,而且不会有AOF可能潜在的Bug,留着做一个万一的手段。
```

### 同步频率设置

- 始终同步 appendfsync always
- 每秒同步 apendfsync everysec，本秒的数据可能丢失
- 不主动同步 appendfsync no 把同步时机交给操作系统

### 重写操作

对AOF进行重写，只关注最后的过程，将记录的指令进行压缩，减少空间占用



# 主从复制

![image-20220501204124933](.\img\image-20220501204009347.png)

主机数据更新后，自动同步到备机的master/slaver机制，Master以写为主，slave以读为主，一般只有一台主服务器

- 负载均衡
- 容灾快速恢复，（服务器挂掉从其他从服务器读



1. 从服务器连上主服务器后，从服务器想主服务器发送进行数据同步消息
2. 主服务器接到从服务器发过来的同步消息，把主服务器数据进行持久化，rdb文件，把rdb文件发送从服务器，从服务器拿到rdb进行读取
3. 每次主服务器进行写操作之后，和从服务器进行数据同步
