---
title: etcd初探
date: 2019-04-18 23:12:44
categories:
- etcd
tags:
- etcd
---
![](https://raw.githubusercontent.com/etcd-io/etcd/master/logos/etcd-stacked-color.png)

本文写于`2019.4.18`，这时etcd项目已经贡献给CNCF，相关迁移工作还在进行中，etcd的文档参考自[coreos.com/etcd](https://coreos.com/etcd/)和[etcd.readthedocs.io](https://etcd.readthedocs.io/en/latest)。

<!-- more -->
## 是什么
etcd是一个分布式`key-value`存储引擎，基于[Raft](https://raft.github.io/)分布式协调一致算法，使用Golang开发。
它具备以下特点：
- 使用Golang开发，具有跨平台、二进制文件小巧和良好的社区支持
- 使用Raft协调一致性算法，易于理解
- 通过分布式锁、leader选举和写障碍保证集群间可靠协调
- 支持集群成员关系动态配置
- 提供高负载下的稳定读/写

## 历史
etcd由Brandon Philips、Alex Polvi和Xiang Li（李想）于2013年创建，当时还是实习生的Xiang Li于`2013.6.6`写下第一行代码并提交。
```shell
commit 20ca21a3f7122cf7caa91cb0e9b9c69be9279950 (tag: 0)
Author: Xiang Li <xiang.li@coreos.com>
Date:   Thu Jun 6 17:43:32 2013 -0700

    store module and unit test for store

```

Brandon Philips认为:
>We wanted Container Linux to be able to essentially reboot one machine at any time, but we wanted people to be able to have application uptime. So we needed to run multiple copies of Container Linux and have some sort of coordination so an entire user application wouldn’t go down at once. This is a pretty well understood coordination problem, and really the way we worked to solve this is through a consensus database.

大致意思是：“我们有多个Container Linux副本，当某个Linux重启时我们希望里面的application的uptime不会被更新，这就需要在多个Container Linux副本中做好协调。我们可以通过一个协调数据库来解决这个一致性问题。”

于是Philips和Xiang Li（当时还是实习生）调研了Google Chubby和Apache Zookeeper，发现不能满足需求。Philips总结了他们的需求：
>We wanted something cloud-capable; something that could be reconfigured on the fly. On the cloud at any point machines are coming and going, IP addresses are changing and you may need to resize the environment. We needed a database that could be reconfigured live without going down. Also you don’t often stand up large infrastructure with gigabytes of RAM and Zookeeper, at the time, required quite a bit of resources of its own. So we wanted something that scaled down well. A further hurdle was the Zookeeper RPC API, Thrift, being rarely used outside of Zookeeper; this meant you couldn’t use common tools like curl to interact with it. We wanted something dynamically reconfigurable that could scale down and was easy to develop against because it used HTTP+JSON. So, Alex Polvi and I sat down and sketched out a readme, and Xiang started working on it in his 2013 summer internship.

大意是：“我们希望这个协调数据库能够在线重配（不重启更改配置）、不占用GB级别的内存、支持HTTP+JSON”

etcd首先应用于CoreOS自家的容器编排工具fleet，同时快速引起社区注意、吸引了第一批用户（应用于DNS、负载均衡和feature flag等）。Kubernetes开源两年后在`2015年`选择etcd作为底层key value存储，这时etcd迎来第二波发展：社区增长了10倍。

etcd使用raft协调一致性算法，该算法由Diego Ongaro提出。Xiang Li开发了Go Raft库，Diego Ongaro也完成了他的博士论文并在论文中感谢了Xiang Li、CoreOS的员工Blake Mizerany和Yicheng Qin。

2018年CoreOS决定把etcd捐赠给CNCF，来更好地促进ercd开源社区发展。Philips有一些新的想法，主要是安全审计，他将协助CNCF从事Kubernetes安全审计相关工作。

## 参考
- [CNCF to Host etcd](https://www.cncf.io/blog/2018/12/11/cncf-to-host-etcd/)
- [The History of etcd](https://coreos.com/blog/history-etcd)
- [etcd contributing guide](https://github.com/etcd-io/etcd/blob/master/CONTRIBUTING.md)