# Truffle使用 #
*Truffle是一个以太坊智能合约开发框架，利用它可以方便地生成项目模板、编译合约、部署合约到区块链、测试合约等等。*

## 一、创建项目 ##
新建一个目录并进入该目录，然后创建项目：

	$ mkdir myContract && cd myContract
	$ truffle init
执行truffle init后，会在当前目录生成一个项目模板，生成的项目目录结构如下:

	myContract
	├── contracts
	│ 	│   
	│   └── Migrations.sol
	├── migrations
	│ 	│
	│   └── 1_initial_migration.js 
	├── test
	│ 
	├── truffle-config.js 
	│ 
	└── truffle.js

contracts下面存放合约代码，migrations里面是部署合约的脚本，test下面是测试脚本，truffle.js是项目的配置文件。

注：truffle版本从3.1.2生成的项目模板中不再包含app目录。

## 二、编写合约 ##
编写合约代码，并把代码文件保存在contracts目录下，默认已经生成了一个示例合约Migrations.sol，如果不需要可以将它删除，本例就使用默认的示例合约文件。

## 三、编译合约 ##
在终端中输入：

	$ truffle compile

如果出现类似如下输出，则编译成功：

	Compiling Migrations.sol...
	Writing artifacts to ./build/contracts

## 四、部署合约 ##
在本机部署合约，先要启动虚拟机testrpc；

修改truffle.js文件

	module.exports = {
	  // See <http://truffleframework.com/docs/advanced/configuration>
	  // to customize your Truffle configuration!
	    networks: {
	      development: {
	        host: '127.0.0.1',
	        port: 8545,
	        network_id: '*'
	      }
	    }
	};
	
还要修改migrations/1\_initial\_migration.js，将deployer.deploy的参数中的合约名改为自己要部署的合约名;

最后部署合约到区块链，在终端中执行：

	$ truffle migrate --reset

如果出现类似如下输出，则部署成功：

	... (省略)
	Running migration: 1_initial_migration.js
	  Deploying ConvertLib...
	  ConvertLib: 0x4d7032160ef9b300fb0cc83cad97819f89e6fc38
	  Linking ConvertLib to MetaCoin
	  Deploying MetaCoin...
	  MetaCoin: 0x3e16298422c552ac794c8a83f4fdae62c6bc2a20
	Saving successful migration to network...
	Saving artifacts...

## 五、测试合约 ##
**进入truffle控制台调试合约**
合约部署成功后，在终端执行truffle console，可以进入Javascript控制台对合约进行调试：

	$ truffle console
	truffle(default)>

在Javascript控制台通过ContractName.deployed()或ContractName.at(contractAddress)获取已部署的合约对象，之后就可以通过该对象调用合约的方法进行调试：

获取已部署的合约对象：

	truffle(default)> var metacoin = Migrations.deployed()

输入.exit可以退出truffle控制台。

**运行单元测试**
Truffle集成了nodejs测试框架Chai(https://github.com/chaijs/chai)，我们可以使用nodejs内置的断言模块assert对合约进行测试。在test/目录下编写合约的测试脚本，然后通过 truffle test 命令执行脚本：

	$ truffle test


