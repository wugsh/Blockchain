# Fabric网络架构

## 1. 网络架构介绍

> 如图所示，fabric网络架构主要包含客户端节点、CA节点、Peer节点、Orderer节点这几个部分。并且fabric架构是安装组织来进行划分当，每个组织都维护不同功能的peer节点。orderer节点属于整个联盟，一般不属于某个组织。

![](../.gitbook/assets/timg.jpg)

其中，各个节点功能如下：

* **CA节点**

  功能：Fabric网络中的成员提供基于数字证书的身份信息。可选。可以用第三方生成的证书；

* **客户端节点**

  功能：与Fabric区块链交互，实现对区块链的操作。常见cli容器及SDK客户端。客户端代表代表最终用户的实体。它必须连接到peer与区块链进行通信。客户端可以连接到其选择的任何peer。客户端创建交易；

* **Peer节点**

  fabric每个组织都包含一个或者多个peer节点。每个节点可以担任多种角色。

  * Endorser Peer（背书节点）

    功能：对客户端发送交易提案时进行签名背书，充当背书节点的角色 。背书\(Endorsement\)是指特定Peer节点执行交易并向生成交易提案\( proposal \)的客户端应用程序返回YES/NO响应的过程。背书节点是动态的角色在链码实例化的时设置背书策略\(Endorsement policy\)，指定哪些节点对交易背书才有效。只有在背书时是背书节点，其他时刻是普通节点。

  * Leader Peer（主节点）

    功能：主要负责与orderer排序节点通信，获取区块及在本组织进行同步。主节点的产生可以动态选举或者指定。

  * Committer Peer（记账节点）

    功能：对区块及区块交易进行验证，验证通过后将区块写入账本中。

  * Anchor Peer（锚节点）

    功能：主要负责与其他组织的锚节点进行通信。  

* **Orderer**

  功能：排序服务节点接收包含背书签名的交易，对未打包的交易进行排序生成区块，广播给Peer节点。排序服务提供的是原子广播，保证同一个链上的节点为接收到相同的消息，并且有相同的逻辑顺序。

## 2. 多通道设计

如图所示，存在3个通道，每个通道包含不同的peer。通道是共识服务提供的一种通讯机制，基于发布-订阅关系，将Peer节点和排序节点根据某个Topic连接在一起，形成一个具有保密性的通讯链路（虚拟），实现业务隔离的要求。  
排序服务提供了供Peer节点订阅的主题（如发布-订阅消息队列），每个主题是一个通道。Peer节点可以订阅多个通道，并且只能访问自己所订阅通道上的交易，因此一个Peer节点可以通过接入多个通道参与到多条链中。  
目前通道分为系统通道（System Channel）和应用通道（Application Channel）。排序服务通过系统通道来管理应用通道，用户的交易信息通过应用通道传递。

![](../.gitbook/assets/tongdao.jpg)

> 在创建通道的时候就定义了多个组织，每个组织一般包含多个peer，这些peer实体的msp证书都可以追溯到组织的证书。并且每个组织还定义了对应锚节点（anchor peer），通过锚节点来和其他组织锚节点通信。其相关配置都在configtx.yaml文件中，即在通道创世块中。并且在后续如果想增加组织/成员/排序节点都可以通过修改配置块来实现。

## 3.交易流程

Fabric区块链的交易分两种，**部署交易**和**调用交易**。  
部署交易把链码部署到Peer节点上并准备好被调用，当一个部署交易成功执行时，链码就被部署到各个背书节点上。  
调用交易是客户端应用程序通过Fabric提供的API调用先前已部署好的某个链码的某个函数执行交易，并相应地读取和写入KV数据库，返回是否成功或者失败。

![](../.gitbook/assets/jiaoyi1.jpg)

```text
交易流程图如图所示，其基本步骤为：

1. 客户端构建交易提案并发送给一个或多个背书节点。

2. 背书节点模拟执行交易及签名
背书节点（endorser）收到交易提案后，验证签名并确定提交者是否有权执行操作。背书节点将交易提案的参数作为输入，在当前状态KV数据库上执行交易，生成包含执行返回值、读操作集和写操作集的交易结果（此时不会更新账本），交易结果集、背书节点的签名和背书结果（支持/拒绝）作为提案的结果返回给客户端。

3. 客户端把交易发送到排序服务节点
客户端收到背书（Endorser）节点返回的信息后，判断提案结果是否一致，以及是否参照指定的背书策略执行，如果没有足够的背书，则中止处理；否则，客户端把数据打包到一起组成一个交易并签名，发送给Orderers。

4. 共识排序，生成新区块及Committer节点确认
Orderers对接收到的交易进行共识排序，然后按照区块生成策略，将一批交易打包到一起，生成新的区块，发送给提交（Committer）节点；提交（Committer）节点收到区块后，会对区块中的每笔交易进行校验，检查交易依赖的输入输出是否符合当前区块链的状态，完成后将区块追加到本地的区块链，并修改世界状态。
```

