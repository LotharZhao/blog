---
title: RocketMQ部署
date: '2024-09-25 10:05:23'
updated: '2024-10-10 11:04:59'
permalink: /post/rocketmq-deployment-1kn9ai.html
comments: true
toc: true
---

# RocketMQ部署

## 修改 broker 配置文件

> conf/broker.conf

```shell
# 所属集群名称
brokerClusterName = DefaultCluster
# 当前brocker名称
brokerName = broker-a
# 0表示Master，>0表示Slave
brokerId = 0
# 删除文件时间点，默认凌晨4点
deleteWhen = 04
# 文件保留时间，默认48小时
fileReservedTime = 48
# Broker的角色：ASYNC_MASTER异步复制、SYNC_MASTER同步双写、SLAVE
brokerRole = ASYNC_MASTER
# 刷盘方式：ASYNC_FLUSH异步刷盘、SYNC_FLUSH同步刷盘
flushDiskType = ASYNC_FLUSH

# 当前broker监听的IP
brokerIP1=127.0.0.1
# 监听端口
listenPort=10911
# 是否允许Broker自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
# 支持消息追踪
traceTopicEnable=true
# 是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
# 在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

# 存储路径
storePathRootDir=/usr/local/rocketmq/store
# commitLog存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
```

## 修改启动脚本

* runserver​

  ```shell
  JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
  ```
* runbroker

  ```shell
  JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m"
  ```

‍

## 启动 namesrv

```shell
nohup bin/mqnamesrv -n 127.0.0.1:9876 > logs/namesrv.log &
```

## 启动 broker

```shell
nohup bin/mqbroker -c conf/broker.conf -n 127.0.0.1:9876 > logs/broker.log &
```

‍

## 启动控制台

```shell
java -Xms256M -Xmx512m -Dfile.encoding=UTF-8 -jar rocketmq-console.jar --rocketmq.config.namesrvAddr=127.0.0.1:9876
```

> 页面地址：http://127.0.0.1:8049

‍

## 安全配置

### 开启 ACL 鉴权

1. brocker 配置文件开启 ACL
2. 修改 `conf/plain_acl.yml` ​文件

   ```yaml
   globalWhiteRemoteAddresses:
     - 10.10.103.*
     - 192.168.0.*

   accounts:
     - accessKey: RocketMQ
       secretKey: 12345678
       whiteRemoteAddress:
       admin: false
       defaultTopicPerm: DENY
       defaultGroupPerm: SUB
       topicPerms:
         - topicA=DENY
         - topicB=PUB|SUB
         - topicC=SUB
       groupPerms:
         # the group should convert to retry topic
         - groupA=DENY
         - groupB=PUB|SUB
         - groupC=SUB

     - accessKey: rocketmq2
       secretKey: 12345678
       whiteRemoteAddress: 192.168.1.*
       # if it is admin, it could access all resources
       admin: true
   ```

‍

### 常见漏洞处理

1. 未授权访问：开启 ACL 鉴权
2. 服务器支持 TLS Client-initiated 重协商攻击(CVE-2011-1473)

   在启动参数中添加：

   ```shell
   JAVA_OPT_EXT: -Djdk.tls.rejectClientInitiatedRenegotiation=true
   ```

‍
