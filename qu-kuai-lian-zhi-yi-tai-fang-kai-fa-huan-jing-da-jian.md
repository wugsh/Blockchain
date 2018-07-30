\#\#区块链之以太坊开发环境搭建

本教程主要针对Debian和Ubuntu系统，实验环境为Debian8.6。

\#\#\#Node.js 环境配置

Node.js是一个Javascript运行环境



&gt;\*\*源码安装Node.js\*\*

&gt;

&gt; 以下部分我们将介绍在Ubuntu Linux下安装 Node.js 。 其他的Linux系统，如Centos等类似如下安装步骤。

&gt;

&gt;1.在 Github 上获取 Node.js 源码：

&gt;

&gt;       $ git clone https://github.com/nodejs/node.git

&gt;

&gt;2.使用 ./configure 创建编译文件，并按照：

&gt;

&gt;       $ cd node

&gt;       $ ./configure

&gt;       $ make

&gt;       $ sudo make install

&gt;3.查看 node 版本：

&gt;

&gt;       $ node --version

     

\*\*安装节点仿真器\*\*



 为了快速开发和测试以太坊 DApp，我们通常使用以太坊节点仿真器来模拟区块链，最流行的节点仿真器就是 Ganache。

&gt;

&gt;\*\*1. 安装Ganache：\*\*

&gt;

&gt;       $ sudo npm install –g ganache-cli

&gt; 验证安装成功：

&gt;

&gt;       $ ganache-cli

&gt;\*\*2. 安装testrpc：\*\*

&gt;

&gt;       $ sudo npm install -g ethereumjs-testrpc

&gt; 验证安装成功使用：

&gt;

&gt;       $ testrpc





 \*\*安装 solidity 编译器solc\*\*



solidity 是开发以太坊智能合约的编程语言 ,编译器是Solc。



&gt;       $ npm install –g solc

&gt;验证安装成功：

&gt;

&gt;       $ solcjs --version

       

\*\*安装 web3\*\*



web3.js是以太坊提供的一个Javascript库，它封装了以太坊的JSON RPC API，提供了一系列与区块链交互的Javascript对象和函数，其中最重要的就是与智能合约交互的API；



&gt;

&gt;       $ npm install web3@^0.20.0

&gt; 验证安装成功：

&gt;

&gt;       $ node -p 'require\("web3"\)'

      

\*\*安装truffle 框架\*\*



Truffle是针对基于以太坊的Solidity语言的一套开发框架，本身基于Javascript。



&gt;

&gt;       $ sudo npm install -g truffle

&gt;验证安装成功：

&gt;

&gt;       $ truffle version





 \*\*安装 webpack\*\*

&gt;

&gt;       $ sudo npm install -g webpack

&gt;

