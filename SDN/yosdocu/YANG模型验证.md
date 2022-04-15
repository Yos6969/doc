# 思路

需要一个简单的方式测试yang模型可用性，要求如下：

- 简单可用
- 支持netconfig协议
- 最好是CS架构

# 方案

- **Netopeer2**是基于NETCONF协议实现网络配置管理的服务器。这是第二代，最初作为[Netopeer 项目](https://github.com/CESNET/netopeer)提供。Netopeer2 基于新一代的 NETCONF 和 YANG 库 - [libyang](https://github.com/CESNET/libyang)和[libnetconf2](https://github.com/CESNET/libnetconf2)。Netopeer2 服务器使用[sysrepo](https://github.com/sysrepo/sysrepo)作为 NETCONF 数据存储实现。

[Netopeer2](https://github.com/CESNET/Netopeer2)

- 使用OpenDayLight框架

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

