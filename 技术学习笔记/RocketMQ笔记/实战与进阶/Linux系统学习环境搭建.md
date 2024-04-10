部署方式可以参考官方文档
[MQ部署方式](https://rocketmq.apache.org/zh/docs/deploymentOperations/01deploy)
# 快速开始
[快速开始](https://rocketmq.apache.org/zh/docs/quickStart/01quickstart)

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