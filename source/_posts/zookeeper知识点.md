---
title: zookeeper知识点
date: 2020-07-18 23:13:06
tags: zookeeper
categories: zookeeper
---

## zookeeper的leader选举机制 ##

1. 集群中每台机器启动时都会先投给自己一票
2. 然后与其它已经启动的机器交流选票决定自己的票投给哪台机器
3. 决定投给哪台机器的规则为先比较zxid，如果zxid一致的话比较myid，投给当前myid最大者
4. 如果有机器的选票数量超过半数机器，则当选为`leader`，其余机器为`flower`

## zookeeper服务器的三种角色 ##

- Leader角色：Leader服务器是zk集群工作的核心，其主要工作有两个：
  1. 事务请求的唯一调度者和处理者，保证集群事务处理的顺序性。
  2. 集群内部各个服务器的调度者

- Follower角色：Follower是zk集群的跟随者，其主要工作有三个：
  1. 处理客户端非事务性请求，转发事务请求给Leader服务器（事务请求都由Leader处理）
  2. 参与事务请求Proposal的投票
  3. 参与Leader选举投票 

- Observer角色：Observer充当观察者角色，观察zk集群的最新状态变化并将这些状态同步过来，对于非事务请求可以进行独立的处理，对于事务请求，则会转发给Leader服务器进行处理，Observer不会参与任何形式的投票，包括事务请求Proposal的投票和Leader选举的投票。场景：集群机器非常多，所以把大部分机器当做观察者（人民群众），一小部分机器参与投票（人大代表），leader只收集小部分机器的投票结果即可，提高了写数据的性能（减少参与投票的机器数）

## zookeeper服务器四种状态 ##

- looking：寻找leader状态，当前服务器处于该状态时，它会认为当前集群中没有leader，因此需要进入leader选举状态
- flowing：跟随者状态，表示当前服务器的角色是Follower角色
- leading：领导者状态，表示当前服务器是Leader
- observing：观察者状态，表示当前服务器角色是Observer

## zookeeper如何保证数据一致性 ##

通过zab协议保持数据一致性,zab协议参照paxos协议实现,主要有以下两点

- 崩溃恢复：没有leader状态时先选leader，崩溃不一定恢复，必须投票数超过半数机器才会选出leader，集群对外正常提供服务
- 正常读写：有leader时正常按照leader广播进行读写，如果发现自己的投票与大部分机器不一致时，崩溃重启（自杀），同步leader的数据，每台机器会有一个待写队列，leader广播可写时写入，不同意写入时回滚（从待写队列移除）