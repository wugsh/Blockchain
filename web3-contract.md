# web3.js与智能合约交互 

*web3.js是以太坊提供的一个Javascript库，它封装了以太坊的JSON RPC API，提供了一系列与区块链交互的Javascript对象和函数，包括查看网络状态，查看本地账户、查看交易和区块、发送交易、编译/部署智能合约、调用智能合约等，其中最重要的就是与智能合约交互的API。*

## 示例合约 ##
本文以下面的MetaCoin合约为例，说明在应用中使用web3.js操作智能合约的方法。

	pragma solidity ^0.4.24;
	contract MetaCoin {
		mapping (address => uint) balances;
		event Transfer(address indexed _from, address indexed _to, uint256 _value);
		function MetaCoin() {
			balances[tx.origin] = 10000;
		}
		function sendCoin(address receiver, uint amount) returns(bool sufficient) {
			if (balances[msg.sender] < amount) return false;
			balances[msg.sender] -= amount;
			balances[receiver] += amount;
			Transfer(msg.sender, receiver, amount);
			return true;
		}
		function getBalance(address addr) returns(uint) {
			return balances[addr];
		}
	}

**这个合约有三个函数：**

- MetaCoin：构造函数，在合约被部署到区块链时执行
- getBalance：返回某账户的余额，它只读数据，不会修改数据
- sendCoin：向另一个账户发送指定数量的MetaCoin，它会改变状态变量balances

启动一个以太坊节点，将上面的合约部署到区块链中，并记录下合约的地址，可以通过truffle部署，具体参考这篇文章。接下来就可以按照下面的步骤在项目中通过web3.js来使用这个合约。

## 添加web3到项目中 ##

首先新建一个Nodejs项目并初始化：

	$ mkdir web3test && cd web3test
	$ npm init
接下来下载web3.js到项目中：

	$ npm install web3 --save
以上命令会将web3.js下载到web3test/node\_modules目录下，其中--save参数会web3.js添加到package.json配置文件中。

## 创建web3对象 ##
要使用web3.js与区块链交互，需要先创建web3对象，然后连接到以太坊节点。
在web3test目录下新建index.js文件，在其中输入以下代码：

	var Web3 = require("web3");
	// 创建web3对象
	var web3 = new Web3();
	// 连接到以太坊节点
	web3.setProvider(new Web3.providers.HttpProvider("http://localhost:8545"));
## 获取已部署的合约实例 ##
要使用智能合约，必须先从区块链中获取到合约实例，获取合约实例需要合约的ABI和合约的地址：

	// 合约ABI
	var abi = *****;
	// 合约地址
	var address = "0xb2cdd356e58280906ce53e1665697b50f88aac56";
	// 通过ABI和地址获取已部署的合约对象
	var metacoin = web3.eth.contract(abi).at(address);
metacoin就是获取到的合约对象实例，此时metacoin对象中就包含了与合约函数同名的Javascript函数，之后就可以通过metacoin对象来调用合约中的函数了。

## 调用合约函数 ##
MetaCoin合约中有两个函数：getBalance和sendCoin，可以通过metacoin对象直接调用这两个函数。

首先来看getBalance，由于getBalance函数只是从区块链中读数据，而不修改数据，因此我们通过在getBalance后面加上.call()的方式调用：

	var account_one = web3.eth.accounts[0];
	var account_one_balance = metacoin.getBalance.call(account_one);
	console.log("account one balance: ", account_one_balance.toNumber());

这里：

- 在getBalance后加上.call()来显式指明用call的方式调用;
- 通过call的方式调用可以得到getBalance函数的返回值;
- 通过call的方式调用的函数只在节点本地虚拟机中执行，不会产生交易，不会花费费用，不会修改数据;

下面来看sendCoin函数，由于sendCoin要修改合约内部的数据，所以要使sendCoin生效，必须要向区块链发送交易，可以在sendCoin后面加上.sendTransaction()来指明这是一笔交易：

	var account_one = web3.eth.accounts[0];
	var account_two = web3.eth.accounts[1];
	// 提交交易到区块链，会立即返回交易hash，但是交易要等到被矿工收录到区块中后才生效
	var txhash = metacoin.sendCoin.sendTransaction(account_two, 100, {from:account_one});
	console.log(txhash);

这里：
 
- 在sendCoin函数后加上.sendTransaction()指明要向区块链发送交易;
- 合约代码中sendCoin函数只有两个参数，而在web3中通过.sendTransaction()调用合约函数的时候需要增加最后一个参数，它是一个javascript对象，里面可以指定from/value/gas等属性，上面的例子用from来指定交易的发送者;
- 上面的调用语句执行后，会向区块链提交一笔交易，这笔交易的发送者是account\_one，接收者是metacoin的地址，交易的作用是以account\_two和100作为参数执行合约的sendCoin函数;
- 函数会立即返回交易的hash，表明交易已经提交到区块链，但是并不知道交易何时处理完成，交易要等到被旷工收录到区块中后才会生效;

## 监听合约事件 ##
当通过.sendTransaction()调用合约的时候，交易会被提交到区块链进行处理，这个处理需要一定的时间，如果需要等交易完成之后再执行其他操作，就必须要知道交易何时完成，那么如何知道交易何时完成呢？可以通过监听合约事件来实现。

在合约中可以定义事件，事件可以带有参数，在合约函数内部完成某些操作时，可以触发事件，向外界传达一些信息。例如，在MetaCoin合约中定义了一个事件叫做Transfer，表示一个转账的事件，它带有三个参数：交易的发送者、接受者、转账数量。在sendCoin函数中，转账成功后就会触发Transfer事件，将对应的参数传给该事件，这样外部监听到事件后，可以取出事件的参数来获得交易发送者、接收者、数量。同时事件中还带有其他信息，比如交易hash等。

在web3中使用事件，要首先获取事件对象，然后监听事件，如果事件发生，就会在回调函数中获取到事件信息：

	// 获取事件对象
	var myEvent = metacoin.Transfer();
	// 监听事件，监听到事件后会执行回调函数
	myEvent.watch(function(err, result) {
	    if (!err) {
	        console.log(result);
	    } else {
	        console.log(err);
	    }
	    myEvent.stopWatching();
	});

	// 输出：
	{ address: '0xb2cdd356e58280906ce53e1665697b50f88aac56',
	  blockNumber: 651,
	  transactionIndex: 0,
	  transactionHash: '0xcc71bc2824cc84d1ee831c46189e3a80cf0af05697ba0370693aa97390c8067b',
	  blockHash: '0x1d53f04206f3926d0f311b1230a9dd0b0c5aadac35b169a6a609e384ab130c6f',
	  logIndex: 0,
	  removed: false,
	  event: 'Transfer',
	  args: 
	   { _from: '0x68b73956d704007514e9257813bdc58cdf3c969a',
	     _to: '0x9c3c1a2f5ef913fac44f0348a78f68d835f3f26e',
	     _value: { [String: '100'] s: 1, e: 2, c: [Object] } } }

从输出可以看出，获取到的事件信息包括事件的参数、事件名、区块号、交易hash等。

通过检测事件中的transactionHash与调用合约函数返回的交易hash是否一致，可以判定某一笔交易是否已完成：

	var account_one = web3.eth.accounts[0];
	var account_two = web3.eth.accounts[1];
	var account_one_balance = metacoin.getBalance.call(account_one);
	console.log("account one balance:", account_one_balance.toNumber());
	var txhash = metacoin.sendCoin.sendTransaction(account_two, 100, { from: account_one });
	var myEvent = metacoin.Transfer();
	myEvent.watch(function (err, result) {
	    if (!err) {
	        if (result.transactionHash == txhash) {
	            var account_one_balance = metacoin.getBalance.call(account_one);
	            console.log("account one balance after sendCoin:", account_one_balance.toNumber());
	        }
	    } else {
	        console.log(err);
	    }
	    myEvent.stopWatching();
	});
	
watch中的回调函数如果被执行，说明事件已被触发，也就是说某笔交易已经处理完，检查交易hash是否一致，可以判定产生这个事件的交易是否是自己发送的交易，如果是，就可以进行其他操作，比如查询最新的余额。