# truffle-contract调用智能合约

_与以太坊的智能合约交互，除了使用web3.js，还可以使用另外一个Javascript库，就是truffle-contract。truffle-contract对以太坊的智能合约做了更好的抽象，相比于web3.js，使用truffle-contract操作智能合约更加方便。_

## truffle-contract具有以下特色：

 1. **同步的交易：**可以确保在交易生效之后再继续执行其他操作 
 2. **返回Promise：**每个封装的合约函数会返回Promise，可以对它进行.then操作，避免了回调地狱（callback hell）问题;
 3. **为交易提供了默认参数：**例如from或gas 
 4. **为每个同步的交易返回logs、交易receipt和交易hash**

## 安装truffle-contract：

首先新建一个Nodejs项目并初始化：

```
$ mkdir truffle-contract-test && cd truffle-contract-test
$ npm init
```

接下来安装truffle-contract：

```
$ npm install web3 --save
$ npm install truffle-contract --save
```

## truffle-contract调用智能合约步骤：

### 1.编写智能合约

在Remix中编写智能合约和部署智能合约，获得合约地址和合约abi。

```
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
```

### 2.初始化合约对象

与web3.js类似，要使用truffle-contract，需要先初始化合约对象，然后连接到一个以太坊节点。在自己的工程新建一个js文件，输入以下代码：

```
//引用web3
var Web3 = require("web3");

//引用truffle-contract
var contract = require("truffle-contract");

// 合约ABI 合约ABI在你的编译JSON文件里面有的
var contract_abi = ******;
// 通过ABI初始化合约对象
var MetaCoin = contract({
    abi: contract_abi
});

// 连接到以太坊节点
var provider = new Web3.providers.HttpProvider("http://localhost:8545");
MetaCoin.setProvider(provider);
```

**接下来，可以使用MetaCoin的以下三个函数：**

* at\(contract\_address\)：通过指定的合约地址获取MetaCoin合约实例；
* deployed\(\)：通过默认的合约地址获取MetaCoin合约实例；
* new\(\)：部署新的合约，并且获取新部署的合约实例；

因为智能合约通过Remix已经部署好，这里就用at的方式获取智能合约的实例，来调用合约。

```
// 账户地址
var account_one = "0x68b73956d704007514e9257813bdc58cdf3c969a";

// 合约地址
var contract_address = "0xb2cdd356e58280906ce53e1665697b50f88aac56";

MetaCoin.at(contract_address).then(function(instance){
    return instance.getBalance.call(account_one);
}).then(function(balance){
    console.log(balance.toNumber());
}).catch(function(err){
    console.log(err);
});
```

首先通过MetaCoin.at\(\)获取合约实例，在.then链式调用中，回调函数会在参数中获取到合约实例instance，接下来就可以在回调函数中使用合约实例来调用getBalance函数，再继续通过.then链式调用，通过回调函数获得getBalance的返回值balance。

**调用sendCoin函数的情况：**

```
// 账户地址
var account_one = "0x68b73956d704007514e9257813bdc58cdf3c969a";
var account_two = "0x9c3c1a2f5ef913fac44f0348a78f68d835f3f26e";

// 合约地址
var contract_address = "0xb2cdd356e58280906ce53e1665697b50f88aac56";

MetaCoin.at(contract_address).then(function(instance){
// 调用sendCoin会向区块链发送一笔交易
    return instance.sendCoin(account_two, 100, {from:account_one});
}).then(function(result){
    // 这个回调函数在交易生效之后才会被执行
    // result对象包含以下三个字段：
    // result.tx  => 交易hash，是一个string
    // result.receipt => 交易执行结果，是一个对象
        // result.logs=> 交易产生的事件集合，是一个对象数组
    console.log(result);
}).catch(function(err){
    console.log(err);
});
```

调用sendCoin会向区块链发送一笔交易，在交易生效之后，才会执行回调函数，回调函数的参数中包含了交易hash、交易执行结果以及交易产生的事件。  
**捕获事件**可以通过result.logs获取交易触发的事件：

```
// 账户地址
var account_one = "0x68b73956d704007514e9257813bdc58cdf3c969a";
var account_two = "0x9c3c1a2f5ef913fac44f0348a78f68d835f3f26e";

// 合约地址
var contract_address = "0xb2cdd356e58280906ce53e1665697b50f88aac56";

MetaCoin.at(contract_address).then(function(instance){
    // 调用sendCoin会向区块链发送一笔交易
    return instance.sendCoin(account_two, 100, {from:account_one});
}).then(function(result){
    // 这个回调函数在交易生效之后才会被执行
    // result.logs是一个数组，数组的每个元素是一个事件对象
    // 通过查询result.logs可以获得感兴趣的事件
    for (var i = 0; i < result.logs.length; i++) {
        var log = result.logs[i];
        if (log.event == "Transfer") {
            console.log("from:", log.args._from);
            console.log("to:", log.args._to);
            console.log("amount:", log.args._value.toNumber());
                break;
        }
    }
}).catch(function(err){
    console.log(err);
});

// 输出：
from: 0x68b73956d704007514e9257813bdc58cdf3c969a
to: 0x9c3c1a2f5ef913fac44f0348a78f68d835f3f26e
amount: 100
```

sendCoin执行完后会触发一个Transfer事件，在回调函数中，通过查询result.logs可以获取到这个事件，进而可以得到事件的参数值。

## 一个完整的例子

```
var Web3 = require("web3");
var contract = require("truffle-contract");

// 合约ABI
var contract_abi = *****;
// 通过ABI初始化合约对象
var MetaCoin = contract({
    abi: contract_abi
});

// 连接到以太坊节点
var provider = new Web3.providers.HttpProvider("http://localhost:8545");
MetaCoin.setProvider(provider);

// 账户地址
var account_one = "0x68b73956d704007514e9257813bdc58cdf3c969a";
var account_two = "0x9c3c1a2f5ef913fac44f0348a78f68d835f3f26e";

// 合约地址
var contract_address = "0xb2cdd356e58280906ce53e1665697b50f88aac56";

var coin;
MetaCoin.at(contract_address).then(function(instance){
    coin = instance;
    // 首先查看账户二的余额
    return coin.getBalance.call(account_two);
}).then(function(balance_of_account_two){
    console.log("Balance of account two is", balance_of_account_two.toNumber());
    // 从账户一向账户二转100个币
    return coin.sendCoin(account_two, 100, {from:account_one});
}).then(function(result){
    console.log("Sent 100 coin from account one to account two.");
    // 再次查看账户二的余额
    return coin.getBalance.call(account_two);
}).then(function(balance_of_account_two){
    console.log("Balance of account two is", balance_of_account_two.toNumber());
}).catch(function(err){
    console.log("Error:", err.message);
});
```



