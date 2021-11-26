## 为什么要给wsl2和本地电脑添加ip？

```  
172.29.6.105wsl -d Ubuntu-20.04 -u root ip addr add 192.168.50.16/24 broadcast 192.168.50.255 dev eth0 label eth0:1
netsh interface ip add address "vEthernet (WSL)" 192.168.50.88 255.255.255.0
```

将这两句写为bat文件，每次开机运行一下，给wsl2和windows设置一个局域网IP

wsl2的网络设置中，修改了wsl中的共享网络模式，改为NAT模式，但每次重启windows都会临时分配一个局域网ip用于 wsl2和windows 的通信，ip地址一直变化，每次都进wsl看一次很麻烦，所以自己设置一个固定的IP地址

## 开机启动wsl 的ssh服务

```   
wsl -d Ubuntu-20.04 -u root service ssh start
```





