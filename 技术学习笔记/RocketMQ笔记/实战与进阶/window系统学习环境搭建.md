部署方式可以参考官方文档
[MQ部署方式](https://rocketmq.apache.org/zh/docs/deploymentOperations/01deploy)
# 安装前的环境准备
JDK1.8、Maven、Git
# 下载二进制程序包
https://rocketmq.apache.org/download
# 配置环境变量
```
变量名：ROCKETMQ_HOME
变量参数：D:\exploit\rocketmq-all-5.1.3   #自己本地的rocketmq地址
```
# 内存分配设置
编辑安装文件夹下中bin目录下的runserver.cmd文件，目的是根据服务器内存大小的实际情况分配内存大小。

```
定位到如下代码 JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m" # 修改　"-Xms -Xmx -Xmn"
参数 JAVA_OPT="${JAVA_OPT} -server -Xms512M -Xmx512M -Xmn256M -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```
还有改bin\\runbroker.cmd
```
"JAVA_OPT=%JAVA_OPT% -server -Xms2g -Xmx2g"
改成
"JAVA_OPT=%JAVA_OPT% -server -Xms256m -Xmx512m"
```
# 启动mqnamesrv、mqbroker.cmd
## 打开CMD界面，cmd命令内容：进入bin目录，先启动namesrv
```
d:
cd D:\source\rocketmq-all-5.1.0-bin-release\bin
start mqnamesrv.cmd
```
![[Pasted image 20240319154701.png]]
如下图表示启动成功
![[Pasted image 20240319154730.png]]
## 然后启动broker
```
start mqbroker.cmd -n 0.0.0.0:9876 -c D:\source\rocketmq-all-5.1.0-bin-release\conf\broker.conf
```
![[Pasted image 20240319161403.png]]
# 安装RocketMQ Dashboard
`RocketMQ Dashboard` 是 RocketMQ 的管控利器，为用户提供客户端和应用程序的各种事件、性能的统计信息，支持以可视化工具代替 Topic 配置、Broker 管理等命令行操作。
## 介绍
### 功能概览[​](https://rocketmq.apache.org/zh/docs/deploymentOperations/04Dashboard#%E5%8A%9F%E8%83%BD%E6%A6%82%E8%A7%88 )

| 面板  | 功能                                |
| --- | --------------------------------- |
| 运维  | 修改nameserver 地址; 选用 `VIPChannel`  |
| 驾驶舱 | 查看 broker, topic 消息量              |
| 集群  | 集群分布，broker 配置、运行信息               |
| 主题  | 搜索、筛选、删除、更新/新增主题，消息路由，发送消息，重置消费位点 |
| 消费者 | 搜索、删除、新增/更新消费者组，终端，消费详情，配置        |
| 消息  | 消息记录，私信消息，消息轨迹等消息详情               |

###  docker 镜像安装
① 安装docker，拉取 `rocketmq-dashboard` 镜像

```
$ docker pull apacherocketmq/rocketmq-dashboard:latest
```

② docker 容器中运行 `rocketmq-dashboard`

```
$ docker run -d --name rocketmq-dashboard -e "JAVA_OPTS=-Drocketmq.namesrv.addr=127.0.0.1:9876" -p 8080:8080 -t apacherocketmq/rocketmq-dashboard:latest
```

提示

`namesrv.addr:port` 替换为 `rocketmq` 中配置的 nameserver 地址：端口号

开放端口号：8080，9876，10911，11011 端口

- 云服务器：设置安全组访问规则
- 本地虚拟机：关闭防火墙，或 `-add-port`
### 源码安装
源码地址：[apache/rocketmq-dashboard](https://github.com/apache/rocketmq-dashboard)

下载并解压，切换至源码目录 `rocketmq-dashboard-master/`

① 编译 `rocketmq-dashboard`

```
$ mvn clean package -Dmaven.test.skip=true
```

② 运行 `rocketmq-dashboard`

```
$ java -jar target/rocketmq-dashboard-1.0.1-SNAPSHOT.jar
```
提示：**Started App in x.xxx seconds (JVM running for x.xxx)** 启动成功

浏览器页面访问：namesrv.addr:8080

关闭 `rocketmq-dashboard` : ctrl + c

再次启动：执行 ②

**tips**：下载后的源码需要上传到 Linux 系统上编译，本地编译可能会报错。