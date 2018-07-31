## 区块链之以太坊开发环境搭建

本教程主要针对Debian和Ubuntu系统，实验环境为Debian8.6。

在配置以太坊开发环境前要安装：git、gcc、make等工具。

```
$ sudo apt-get install git  build-essential
```

### Node.js 环境配置

Node.js是一个Javascript运行环境

> **源码安装Node.js**
>
> 以下部分我们将介绍在Ubuntu Linux下安装 Node.js 。 其他的Linux系统，如Centos等类似如下安装步骤。
>
> 1.在 Github 上获取 Node.js 源码：
>
> ```
>   $ git clone https://github.com/nodejs/node.git
> ```
>
> 2.使用 ./configure 创建编译文件，并按照：
>
> ```
>   $ cd node
>   $ ./configure
>   $ make
>   $ sudo make install
> ```
>
> 3.查看 node 版本：
>
> ```
>   $ node --version
> ```

**安装节点仿真器**

为了快速开发和测试以太坊 DApp，我们通常使用以太坊节点仿真器来模拟区块链，最流行的节点仿真器就是 Ganache。

> **1. 安装Ganache：**
>
> ```
>   $ sudo npm install –g ganache-cli
> ```
>
> 验证安装成功：
>
> ```
>   $ ganache-cli
> ```
>
> **2. 安装testrpc：**
>
> ```
>   $ sudo npm install -g ethereumjs-testrpc
> ```
>
> 验证安装成功使用：
>
> ```
>   $ testrpc
> ```

**安装 solidity 编译器solc**

solidity 是开发以太坊智能合约的编程语言 ,编译器是Solc。

> $ npm install –g solc  
> 验证安装成功：
>
> ```
>   $ solcjs --version
> ```

**安装 web3**

web3.js是以太坊提供的一个Javascript库，它封装了以太坊的JSON RPC API，提供了一系列与区块链交互的Javascript对象和函数，其中最重要的就是与智能合约交互的API；

> $ npm install web3@^0.20.0  
> 验证安装成功：
>
> ```
>   $ node -p 'require("web3")'
> ```

**安装truffle 框架**

Truffle是针对基于以太坊的Solidity语言的一套开发框架，本身基于Javascript。

> $ sudo npm install -g truffle  
> 验证安装成功：
>
> ```
>   $ truffle version
> ```

**安装 webpack**

> $ sudo npm install -g webpack


