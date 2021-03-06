# 6 Zookeeper
## 6.1 概述

Zookeeper 是一个分布式开源框架，有两个最基本的功能 : 存储小数据和监听机制，一般用于分布式锁、分布式集群管理、集群选主。

## 6.2 集群特性

1、集群中每个服务器保存一份相同的数据副本；

2、一份数据会在所有机器上都保存一份；

3、全局有序和偏序 : 全局有序是指如果在一台机器上消息 a 在消息 b 之前被发布，则在集群中所有机器上消息 a 都在消息 b 之前被发布。偏序是指如果消息 b 被同一个发送者在消息 a 之后发布，则消息 a 永远排在消息 b 前面。

4、一次数据更新要么成功（半数以上节点成功），要么失败，不存在中间状态。

## 6.3 集群角色

**Leader :** Zookeeper 集群工作的核心，事务请求（写操作）的唯一调度和处理者，保证集群事务处理的顺序性，集群内部各个服务器的调度者。

**Follower :** 处理客户端非事务（读操作）请求，转发写清求个 Leader，参与集群 Leader 选举投票。

**Observer :** 观察 Zookeeper 集群的最新状态并将这些状态同步过来，对于非事务性请求可以独立处理，对于事务性请求，则会转发给 Leader 进行处理，不会参与任何形式的投票只提供非事务服务，通常用于在不影响集群性能、处理能力的前提下提升集群的非事务处理能力。

## 6.4 典型应用
### 6.4.1 数据发布于订阅（配置中心）

发布者将数据发布在 zk 节点上，供订阅者动态获取数据，实现配置信息的集中管理和动态更新。

### 6.4.2 集群选主

在一个集群中有一个 active 节点，多个 standby 节点，当一个节点挂掉时，可以选出一个 standby 节点，并将其状态更改为 active，从而实现集群的高可用。

### 6.4.3 分布式锁

所有客户端要想获得执行权，就需要在 /locked 下创建一个节点，如果这个节点已经创建，那么用户就不能在创建节点，无法获得执行权，这样就保证了在同一个时刻只有一个客户端拥有执行权，当执行完成后，客户端会删除所创建的节点，这样所有的客户端就会争抢去创建节点，从而实现分布式锁。

## 6.5 选举机制

服务器 1 启动 -> 给自己投票 -> 票数为 1，没有过半 -> 处于观望状态

服务器 2 启动 -> 给自己投票 -> 服务器 1 的编号小于服务器 2 的编号，服务器 1 屈服，将投票给服务器 2 -> 票数为 2，没有过半 -> 处于观望状态

服务器 3 启动 -> 给自己投票 -> 服务器 1,2 的编号都小于 3，服务器 1,2 屈服，将投票给服务器 3 -> 票数为 3，过半 -> 服务器 3 胜出

服务器 4 启动 -> 给自己投票 -> 票数为 1，但服务器 3 已经胜出 -> 服务器 4 屈服

服务器 5 启动 -> 给自己投票 -> 票数为 1，但服务器 3 已经胜出 -> 服务器 5 屈服