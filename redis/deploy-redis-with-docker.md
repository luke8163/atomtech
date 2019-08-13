# 使用Docker搭建Redis

## 前言

目前来说，高可用(主从复制、主从切换)redis集群有两种方案，一种是redis-sentinel，只有一个master，各实例数据保持一致；一种是redis-cluster，也叫分布式redis集群，可以有多个master，数据分片分布在这些master上。
  本文介绍基于docker和redis-sentinel的高可用redis集群搭建，大多数情况下，redis-sentinel也需要做高可用，这里先对redis搭建一主二从环境，另外需要3个redis-sentinel监控redis master。

> 很显然，只使用单个redis-sentinel进程来监控redis集群是不可靠的，由于redis-sentinel本身也有single-point-of-failure-problem(单点问题)，当出现问题时整个redis集群系统将无法按照预期的方式切换主从。官方推荐：一个健康的集群部署，至少需要3个Sentinel实例。另外，redis-sentinel只需要配置监控redis master，而集群之间可以通过master相互通信。


## redis-sentinel

redis-sentinel作为独立的服务，用于管理多个redis实例，该系统主要执行以下三个任务：

- 监控 (Monitor): 检查redis主、从实例是否正常运作
- 通知 (Notification): 监控的redis服务出现问题时，可通过API发送通知告警
- 自动故障迁移 (Automatic Failover): 当检测到redis主库不能正常工作时，redis-sentinel会开始做自动故障判断、迁移等操作，先是移除失效redis主服务，然后将其中一个从服务器升级为新的主服务器，并让失效主服务器的其他从服务器改为复制新的主服务器。当客户端试图连接失效的主服务器时，集群也会向客户端返回最新主服务器的地址，使得集群可以使用新的主服务器来代替失效服务器


### sentinel.conf
`sentinel.conf`是启动redis-sentinel的核心配置文件，可以从官网下载：
```shell
wget http://download.redis.io/redis-stable/sentinel.conf
```