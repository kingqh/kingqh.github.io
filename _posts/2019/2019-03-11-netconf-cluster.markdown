---
layout:     post
title:      "基于ODL的NETCONF分布式下发工程实现"
date:       2019-03-11
author:     "Kingqh"
header-img: "img/post-bg-odl.jpg"
tags:
    - ODL
---

# 基于ODL的NETCONF分布式下发工程实现 #
---

## 导语 ##

OpenDaylight简称(ODL)是目前使用比较广泛的SDN软件开发平台。它提供了一套丰富的南向协议包括OpenFlow、Netconf、SNMP、PCEP等标准协议的插件。本文主要介绍ODL南向插件Netconf的分布式工程应用和实践，为以后控制器应用开发提供高效的配置管理功能。

## 背景介绍 ##

### Netconf组件 ###

ODL对Netconf协议的支持还是比较全面的，可以通过将支持Netconf Server的设备作为MountPoint挂载到ODL的拓扑，然后使用MountPoint节点对应的DataBroker服务的read、put、merge和delete方法，对设备配置进行增删改查操作。数据库操作的LogicalDatastoreType对应的两种类型CONFIGURATION和OPERATIONAL分别对应Netconf的两种数据类型即状态数据和配置数据。

那么ODL是如何做到将Netconf设备作为挂载点后，直接操作其DataBroker数据库来实现对设备做Netconf协议控制呢？

1.ODL的MD-SAL内核与Netconf协议支持同样的建模语言yang，在一定程度上可以方便Netconf协议的无缝开发。ODL内部正是使用了yang建模实现了Netconf设备的报文结构定义，为不同厂商的设备管理提供方便得适配功能。

2.ODL提供了分布式3PC提交方式的数据库服务DataBroker，用来处理Netconf的配置数据的操作。DataBroker提供的的read、put、merge和delete对应Netconf操作类型get-config和edit-config。

3.同时，ODL提供了Netconf-Client模块实现协议客户端功能。下图描述了ODL如何通过DataBroker操作向设备下发配置的流程。

![aaa]({{ site.url }}/img/in-post/2019-03-11/DB-Netconf.png)

其中DataObject是基于yang模型生成的JAVA对象，而且此yang模型是设备所支持的配置数据结构。有两种方式可以获取到设备支持的yang模型：一种是通过NETCONF Monitoring功能，使用get-schema获取到所有支持的yang；另一种方式就是从官网上下载或者设备商提供。

上图只是描述了下发配置的流程，dataBroker的put对应的是netconf的下发功能，具体各个操作的对应关系列表如下：

![table1]({{ site.url }}/img/in-post/2019-03-11/table1.png)

事实上，dataBroker的两种数据库类型分别对应Netconf的两种配置类型数据：

![table2]({{ site.url }}/img/in-post/2019-03-11/table2.png)

## EntityOwnershipService组件 ##

EntityOwnershipService服务主要作用是提供实体所有权的分配。集群中所有成员通过向EntityOwnershipService注册所管理的Entity以及EntityOwnershipListener监听器。Entity包装了所有管理的实体Id,EntityOwnershipListener监听器提供ownershipChanged方法，能够在选出实体的owner后，触发监听器。

目前EntityOwnershipService支持的实体所有权的分配策略有两种：FirstCandidateSelectionStrategy和LeastLoadedCandidateSelectionStrategy。顾名思义，前者指的是将实体所有权分配给最先注册实体的成员，而后者是将实体所有权分配给目前拥有实体数量最少的成员。两种策略应用的场景不同，达到的负载均衡效果不同。FirstCandidateSelectionStrategy依靠的是成员注册的随机性保证实体分配到不同集群节点上，而LeastLoadedCandidateSelectionStrategy能够保证在所有成员节点都注册后，实现实体的均匀负载分担。


## Routed Rpc组件 ##

ODL的MD-SAL内核提供了Rpc远程调用服务，主要包括Global Rpc和Routed Rpc两种类型，Global Rpc适合在单实例环境中使用，而Routed Rpc可以实现分布式系统多实例间的服务注册和调用。

Routed Rpc主要通过Gossip协议将Rpc服务实现泛洪到分布式系统的所有节点，这样未实现Rpc的节点可以通过获取到对应的实例Id,通过MD-SAL路由到对应的节点上。Gossip协议是一种用随机的方式将信息散播到有界的网络中，最终实现整网状态一致性的算法。


![ccc]({{ site.url }}/img/in-post/2019-03-11/Routed-Rpc.png)

如上图所示：节点MD-SAL1实现RPC1服务，并且将RPC1服务对应的实例ID通过Gossip发布出去。节点MD-SAL2和MD-SAL3和保存了ID1和MD-SAL1的RPC1对应关系，当有用户访问此服务时，便会路由会MD-SAL1节点调用RPC1。这里的实例Id是Rpc注册服务时所传的InstanceIdentifier，用来记录服务路径的标识。


## NETCONF分布式下发实现 ##

![ddd]({{ site.url }}/img/in-post/2019-03-11/NC-Cluster.png)

1.NC-M是由NC集群服务通过eos(EntityOwnershipService)服务选举出的owner,对所有netconf设备进行管理。主要提供register-device和unregister-device两个北向接口，分别用来实现netconf设备的加入和离开。

2.当有设备加入或者离开netconf集群，NC-M将从datastore数据存储中添加或者删除对应节点信息，此时集群中NC成员监听到datastore变化，触发向eos服务注册或者取消注册设备的entity实体。

3.成员中所有的NC均将设备注册给eos服务后，eos通过配置好的策略从所有NC中选择出设备所注册实体对应的owner成员。

4.被选择为owner的NC成员将会和对应注册实体的设备进行netconf会话建立，并且保持keep-alive长连接，同时通过在NC成员上注册对应设备Id的业务配置下发Routed Rpc服务，进行后续的netconf业务下发。

上图表示的是三个不同类型设备CISCO、JUNIPER和HW，通过Netconf集群负载分担，将不同设备的netconf会话建立、业务下发等功能分发到不同成员NC1、NC2和NC3。NC1、NC2和NC3节点是安装了南向ODL Netconf插件的控制器节点，负责Netconf会话的维护以及业务配置下发功能。

## NETCONF分布式下发测试 ##

以下表格是分别对三百台设备进行的单台集中式控制器以及三台分布式控制器集群的测试数据。表中的控制器节点配置均为2G内存2*2核CPU。

![table3]({{ site.url }}/img/in-post/2019-03-11/table3.png)

由上表可以看出，分布式下发相比集中式下发性能优势很明显，而且具有容灾备份的功能。

## 总结 ##

本文主要介绍了ODL的三个组件，Netconf、EntityOwnershipService以及Routed Rpc。基于ODL的这几个组件，我们实现了NETCONF的分布式下发，以及集群容灾功能。目前，该方案已经在我们的DCI骨干网络中使用，用来作为控制器的南向插件。
