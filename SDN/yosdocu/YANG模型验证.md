# 思路

需要一个简单的方式测试yang模型可用性，要求如下：

- 简单可用
- 支持netconfig协议
- 最好是CS架构

# 方案

- **Netopeer2**是基于NETCONF协议实现网络配置管理的服务器。这是第二代，最初作为[Netopeer 项目](https://github.com/CESNET/netopeer)提供。Netopeer2 基于新一代的 NETCONF 和 YANG 库 - [libyang](https://github.com/CESNET/libyang)和[libnetconf2](https://github.com/CESNET/libnetconf2)。Netopeer2 服务器使用[sysrepo](https://github.com/sysrepo/sysrepo)作为 NETCONF 数据存储实现。

[Netopeer2](https://github.com/CESNET/Netopeer2)

- 使用OpenDayLight框架，没弄明白

#实施

##Netopeer2方案

安装依赖库

- Netopeer2
  - libyang
    - libpcre2
  - libnetconf2
    - libyang
    - libssh
    - OpenSSL
    - cmocka -可选
  - sysrepo
    - libyang



```bash
git clone https://github.com/CESNET/libyang.git

#libyang

	#pcre2
wget https://github.com/PhilipHazel/pcre2/releases/download/pcre2-10.39/pcre2-10.39.tar.gz
tar -xvf pcre2-10.39.tar.gz
cd pcre2-10.39
./configure   #--enable-utf, --enable-unicode-properties  貌似没用

cd libyang
mkdir build
cd build
make 
sudo make install

	#libssh
$ wget https://git.libssh.org/projects/libssh.git/snapshot/libssh-0.9.0.tar.gz
$ tar -xvf libssh-0.9.0.tar.gz
$ cd libssh; mkdir build; cd build
$ cmake ..
$ make

# make install
	#openssl 大部分linux发行版自带
	#cmocka
sudo apt-get install libcmocka-dev

#netconf2
cd netconf2
mkdir build
cmake ..
make 
sudo make install

#sysrepo
git clone git@github.com:sysrepo/sysrepo.git
$ mkdir build; cd build
$ cmake ..
$ make
# make install

#netopeer2
$ mkdir build; cd build
$ cmake ..
$ make
$sudo  make install
#可能需要解决 /usr/loocal/lib库的加载问题
```

### 测试

如果linux下打开全是^M,需要考虑windows下的回车换行符问题,windows下为/r/n linux下为/n

```
cat filename |tr -d '\r' > newfile
```

yang转yin

```
yanglint --format=YIN ./ietf-yang-xxx.yang
```

yang转yang tree

```
 yanglint --format=TREE raisecom-vxlan.yang
```

yang+xml转json

```
yanglint    --format=json  --type=config  --output=data.json ./ietf-yang-push@2019-09-09.yang ./data.xml
```



```bash
netopeer2-cli #启动客户端
netopeer2-server -v2 -d #启动服务器 -d打印输出 -v2 输出所有信息
[INF]: LY: Loading schema from "/home/yos114514/netopeer2-2.1.16/sysrepo/build/repository/yang/sysrepo-monitoring@2021-07-29.yang" file.
[INF]: LY: Implemented module "ietf-datastores@2018-02-14" is not used for import, revision "2018-02-14" is imported instead.
[INF]: LY: Searching for "sysrepo-plugind" in "/home/yos114514/netopeer2-2.1.16/sysrepo/build/repository/yang".

#客户端连接
connect  # localhost 830
[INF]: LN: Received an SSH message "request-service" of subtype "ssh-userauth".
[INF]: LN: Received an SSH message "request-auth" of subtype "none".
[INF]: LN: Received an SSH message "request-auth" of subtype "interactive".


客户端支持 get get-config edit-config操作
subscribe 此操作启动一个事件通知订阅，该订阅
      将向发起者发送异步事件通知
      命令直到订阅终止。
```



### sysrepo

多数linux的应用程序需要有配置，配置文件的保存和读写通常的实现方式是通过操作文件来完成的。各应用程序都自定配置文件的格式，格式风格存在诸多差异。

Sysrepo是一个基于YANG模型的配置和操作数据库，为应用程序提供一致的操作数据的接口，解决了配置读写困难的问题。应用程序使用YANG模型来建模，这样就可以利用YANG模型完成数据合法性的检查，保证的风格的一致，不需要应用程序直接操作配置文件了。


SYSREPO数据库它提供了以下特性:

- 模型配置文件和状态数据的集中存储
- 应用程序可以通过XPATH访问配置
- 支持启动、运行和临时数据存储
- 支持事务，符合ACID
- 根据YANG模型，进行数据一致性和约束的检查
- 没有单一故障点，应用程序不需要运行任何其他进程来访问其配置

```
sysrepoctl -l #查看已安装的模块
```

订阅实际上就是回调函数。
例如我们的程序告诉sysrepo，我们要订阅/net/eth0/ip这个xml地址，当有人发消息给sysrepo，写这个路径时，sysrepo就会告诉我们这个xml发生变化了，我们就执行实际的操作。

 sysrepoctl is a command-line tool for manipulation of YANG schemas in sys‐
       repo. It can list the currently installed schemas and add, remove, or mod‐
       ify them.



```
sudo sysrepoctl -i xxx.yang #安装yang模块
```



## 客户端连接测试

使用YumaBench--图形netconfig客户端

- 基于 yangcli-pro 客户端，包括命令行功能、帮助和补全
- NETCONF over SSH/TLS 密码和证书认证
- 服务器 YANG 数据动态显示，适应：
  - 杨对象
  - 可选功能
  - 增强
  - 偏差
- 放大大型 YANG 数据树以创建您需要的特定视图
- 通过搜索对象和数据找到“大海捞针”
- 可定制的 YANG 对象着色以优化识别
- 无需xml
- 过滤视图

```bash
ssh-copy-id yos114514@10.112.3.211 -p14942 #拷贝公钥至服务器
```

![yumbench](.\img\yumbench.png)
