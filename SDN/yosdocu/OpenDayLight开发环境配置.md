# opendaylight开发者环境配置

在 OpenDaylight 控制器上开发应用程序

[官方文档](https://docs.opendaylight.org/en/stable-phosphorus/developer-guides/developing-apps-on-the-opendaylight-controller.html#)

我这边使用环境如下

- vmware的ubuntu 20.04镜像
- java版本为1.11, 设置好JAVA_HOME和PATH
- maven版本为 3.8.5, apt install 的版本不够，可能要自己到官网下release, 解压后设置好MAVEN_HOME和PATH

mvn提取命令参照opendaylight官方文档(狗都不看)，如下：

```bash
mvn archetype:generate -DarchetypeGroupId=org.opendaylight.controller -DarchetypeArtifactId=opendaylight-startup-archetype \
-DarchetypeCatalog=remote -DarchetypeVersion=1.3.0-Carbon
```

进入拉取的文件夹下,执行

```
mnv clean install 
```

报错----缺少依赖

上stackoverflow看了一圈

```bash
#mvn archetype:generate -DarchetypeGroupId=org.opendaylight.controller -DarchetypeArtifactId=opendaylight-startup-archetype -DarchetypeRepository=http://nexus.opendaylight.org/content/repositories/opendaylight.release -DarchetypeVersion=1.9.3

mvn archetype:generate -DarchetypeGroupId=org.opendaylight.controller -DarchetypeArtifactId=opendaylight-startup-archetype -DarchetypeRepository=http://nexus.opendaylight.org/content/repositories/opendaylight.release/ -DarchetypeCatalog=remote -DarchetypeVersion=1.5.1

```

生成成功，进入拉去下来的文件夹

```
mvn clean install -DskipTests -Dmaven.javadoc.skip=true -Dcheckstyle.skip=true
```



生成成功后，应该可以参照，[ncmount](https://wiki-archive.opendaylight.org/view/Controller_Core_Functionality_Tutorials:Tutorials:Netconf_Mount) 进行设备挂载,

- 发现连接到控制器的 Netconf 设备
- 在附加的 Netconf 设备上执行读写操作
- 构建一个可以与 Netconf 设备一起使用的应用程序（即它显示依赖关系和
- 使用 YangUI 在附加设备上进行操作
