#fabric1.4 入门
主要是体验了下`fabric`基本功能——快速搭建起网络环境，并利用自带`sample`，体会了一下链码的部署与调用，为后续逐步深入其内部原理和链码开发做一个铺垫。
##1 名词解释
![](https://i.imgur.com/5NswGJB.png)
![](https://i.imgur.com/WDnw6mw.png)

##1.1 环境准备
**系统：**

		Ubuntu16.04/18.04

###1.1.1 搭建过程
**go安装：**安装的是1.10.4版本，可直接安装  `sudo apt-get install golang`

**docker安装：**
我安装的是18.09.4版本, 可直接安装`sudo apt-get install docker`

**docker-compose安装：**
我安装的是1.23.2版本，可直接安装`sudo apt-get install docker-compose`

**nodejs安装：**

安装nodejs（要求版本8.9.x）和npm（要求版本5.6.x);
	
	sudo apt-get install nodejs
	sudo npm install n
	sudo n 8.9.4    //调整node版本到8.9.4
**安装git：** `sudo apt-get install git`


##2 下载相关镜像文件
###2.1 下载fabric源码：
`git clone https://github.com/hyperledger/fabric.git
`

下载完成后会得到一个`fabric`文件夹，进入`fabric/scripts`目录可以看到一个`bootstrap.sh`脚本（注意刚开始是没有`fabric-samples`这个文件夹的，是执行脚本后生成的）。
直接执行`bootstrap.sh`脚本，就会自动进行`fabric`相关镜像的下载。

##3 构建网络
下面基于`fabric-samples`提供的BYFN(build your first network)，来快速的构建我们第一个超级账本`fabric`网络，以此来熟悉整个运行过程。 
###3.1 生成配置
进入`fabric/scripts/fabric-samples/first-network/`，执行以下命令：

    ./byfn.sh -m generate -c jschannel

####3.1.1 命令说明
- 这条命令也可以不用执行，在使用`up`命令启动网络时，若发现未生成所需配置，会自动进行执行。

- 该条命令将根据配置文件`（crypto-config.yaml）`生成初始化配置，包含`Peer`节点、排序服务节点的`MSP`证书
- 根据配置文件`configtx.yaml`生成创世区块等;

####3.1.2 步骤说明：
**（1）生成证书**
> 使用`cryptogen`工具为组织`org1`和`org2`生成`MSP`证书，`MSP`证书是超级账本网络实体的身份标识，实体在通信和交易时，使用证书进行签名和验证。成功执行后，证书文件将会放到`crypto-config`目录中。
 
**（2）创建创世块**
>为排序服务生成创世区块，成功执行后，会在`channel-artifacts`目录下生成创世区块文件：`genesis.block`。

**（3）创建通道**
> 生成通道配置文件`channel.tx`，成功执行后，会在`channel-artifacts`目录下生成通道配置`channel.tx`。

**（4）生成锚节点配置**
>生成`Org1MSP`和`Org2MSP`的锚节点配置，成功执行后，生成的锚点配置文件`Org1MSPanchors.tx`和`Org2MSPanchors.tx`同样位于目录`channel-artifacts`中。

####3.1.3 配置输出
综上，命令执行完后，将会在当前目录生成两个文件夹，内容分别如下：

- 创世区块目录`（channel-artifacts）`；
![](https://i.imgur.com/nSSHAEp.png)
- 证书目录`（crypto-config）`
>该目录存放生成排序服务节点和`Peer`节点的`MSP`证书文件;

###3.2 启动网络
执行以下命令启动网络

`./byfn.sh -m up -c jschannel`

通过`top`命令可以看到此时`fabric`网络`peer`节点的运行情况。
####3.2.1 命令说明

- 执行该命令后，首先会根据`docker-compose-cli.yaml`配置文件启动超级账本`fabric`网络

- 其中包含了1个排序服务节点、4个`Peer`节点，以及1个命令行容器`cli`
####3.2.2 查看容器
使用`docker ps`命令查看容器启动情况

###3.3 关闭网络命令
执行以下命令关闭提供的`fabric-samples`中的`first-network`网络

    ./byfn.sh -m down

###3.4 链码调用
####3.4.1 执行命令

> 在**启动网络**使用的`$ sudo ./byfn.sh -m up -c jschannel`中，启动的
`cli`容器会调用`scripts/script.sh`脚本，该脚本会进行通道创建、`Peer`节点加入通道、安装和实例化链码等，并进行链码的调用和查询。下面我们对命令进行进一步的说明。

####3.4.2  命令说明
`scripts/script.s`h中执行的具体命令如下，每个命令都对应一个`shell`脚本函数；
![](https://i.imgur.com/u9kLcsl.png)

####3.4.2 步骤说明
**（1）创建通道**
> 进入`cli`容器，执行创建通道命令。成功执行后，将在`cli`容器的当前路径下生成通道配置`jschannel.block`，加入通道操作时需要使用。

![](https://i.imgur.com/FT2H8ZT.png)

**（2）4个peer节点依次加入通道**

> 通过在`cli`容器中设置不同的环境变量，设置连接的`Peer`节点和`MSP`等，依次将4个`Peer`节点加入通道。

![](https://i.imgur.com/ilx5cTQ.png)

**（3）更新组织的锚节点**

![](https://i.imgur.com/N1j2XOq.png)

**(4) 在`Peer`节点（`peer0.org1`和`peer0.org2`）上安装链码**

> 应用程序通过链码执行智能合约的功能，需要先在`Peer`节点上安装链码，然后在通道上实例化链码;

![](https://i.imgur.com/zxR2uvp.png)

**（5）在`Peer`节点（`peer0.org2`）上实例化链码**
> 实例化链码只需要执行一次，这条命令比较复杂，`-c`：指定初始化参数；`-P`：指定了背书策略，链码的每个交易需要由`Org1MSP`和`Org2MSP`两个组织都进行背书签名，才能通过背书策略，继续进入后续交易流程。成功执行后，会启动链码容器`mycc`。

![](https://i.imgur.com/akEG4Xu.png)

**（6）在Peer节点（peer0.org1）上执行链码查询a的值**
![](https://i.imgur.com/EFUItv4.png)


**（7）在Peer节点（peer0.org1）调用链码，从a转10到b**

![](https://i.imgur.com/OzW16jo.png)

**（8）在Peer节点（peer1.org2）安装链码**
![](https://i.imgur.com/7EDZdue.png)


**（9）在Peer节点（peer1.org2）执行链码查询a的值**
![](https://i.imgur.com/Z4T1eUu.png)

###3.5 关闭网络
执行以下命令关闭提供的`fabric-samples`中的`first-network`网络

    ./byfn.sh -m down

##4 手动体验fabric部署
先构建网络，也就是通过`bootstrap.sh`脚本下载完成相关镜像文件。
###4.1 部署超级账本网络
先下载`fabric-samples`文件

    git clone https://github.com/hyperledger/fabric-samples.git

进入`basic-network`目录，利用`docker-compose`启动容器

![](https://i.imgur.com/dSLvUDj.png)

可以用`docker ps`查看容器启动情况：

![](https://i.imgur.com/o7XAdcS.png)

**切换到管理员用户再创建通道和加入通道：**

切换环境到管理员用户的`MSP`，进入`peer`节点容器`peer0.org1.example.com`

    docker exec -it -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com bash

创建通道
    
    root@8bce6cf7abcd:/opt/gopath/src/github.com/hyperledger/fabric# peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx

加入通道

    root@8bce6cf7abcd:/opt/gopath/src/github.com/hyperledger/fabric# peer channel join -b mychannel.block
    
退出`peer`节点容器`peer0.org1.example.com`

    root@8bce6cf7abcd:/opt/gopath/src/github.com/hyperledger/fabric# exit

进入`cli`容器安装链码和实例化

    [root@master basic-network]# docker exec -it cli /bin/bash

给`peer`节点`peer0.org1.example.com`安装链码

    root@0c6f50b96708:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode install -n mycc -v v0 -p github.com/chaincode_example02

这一步可能会报下面的错：
根据错误可知，是`/opt/gopath/src/github.com/chaincode_example02`路径下没有需要的`chaincode_example02.go`文件，通过下面的命令找到了此文件
![](https://i.imgur.com/pHZFqz3.png)

找到后将此文件复制到`/opt/gopath/src/github.com/chaincode_example02`路径下即可解决（注意要在`cli`容器用户下进行复制）。

**实例化链码**

    root@0c6f50b96708:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n mycc -v v0 -c '{"Args":["init","a","100","b","200"]}'


**链码调用和查询**

链码实例化之后就可以查询初始值了，同样是在`cli`容器中进行
`root@0c6f50b96708:/opt/gopath/src/github.com/chaincode_example02# peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'`

调用链码，从“a”转移10到“b”
`root@0c6f50b96708:/opt/gopath/src/github.com/chaincode_example02# peer chaincode invoke -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}`'

再次查询“a”和“b”的值

    root@0c6f50b96708:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
    90
    root@0c6f50b96708:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode query -C mychannel -n mycc -c '{"Args":["query","b"]}'
    210
