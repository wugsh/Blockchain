# 案例1:  使用Remix发行自己的代币

Remix [https://remix.ethereum.org/](https://remix.ethereum.org/) 这是最简单方便的智能合约开发环境，直接在浏览器里编写、调试智能合约。

**step 1. 开发环境**

> 浏览器打开：[https://remix.ethereum.org/](https://remix.ethereum.org/)

**step 2.新建合约**

> 点击左上角“+”号创建一份新的合约，只能以.sol为后缀，如图：
>
> > ![](/assets/选区_001.png)

**step 3.  编写代币合约代码**  
使用solidity编写合约,这个合约只做两件事：

> > 创造代币：发起合约时创造指定数量的代币，给创建者所有初始代币.
> >
> > 转移代币：转移指定数量的代币到指定的Ethereum帐户

```
pragma solidity ^0.4.20;

contract MyToken {
    /* This creates an array with all balances */
    mapping (address => uint256) public balanceOf;

    /* Initializes contract with initial supply
    tokens to the creator of the contract */

    function MyToken( uint256   initialSupply) public {
        // Give the creator all initial tokens
        balanceOf[msg.sender] = initialSupply;
        }

    /* Send coins */
    function transfer(address _to, uint256 _value) public {
        // Check if the sender has enough
        require(balanceOf[msg.sender] >= _value);
        // Check for overflows
        require(balanceOf[_to] + _value >= balanceOf[_to]);
        // Subtract from the sender
        balanceOf[msg.sender] -= _value;
        // Add the same to the recipient
        balanceOf[_to] += _value;
        }

}
```

**step 4.使用Remix编译合约**

> 点击文档右边的 “Start to compile” 开始编译，旁边没有跳出红色的 ERROR 就代表编译成功。
>
> > ![](/assets/选区_002.png)

**step 5.在本地部署合约**

> 1.设置环境选项
>
> > 点击右边的RUN，Environment 选择 JavaScript VM。
> >
> > ![](/assets/选区_003.png)
> >
> > Remix有三种不同的环境选项，可用于部署/测试Solidity合约：JavaScript VM，Injected Web3和Web3 Provider。
> >
> > JavaScript VM选项，这是一个Remix自己的内部沙盒，它不连接到MainNet，TestNet或任何专用网络。可用于简单测试和快速挖掘。
> >
> > 这个环境提供5个测试账号，每个账号有100Eth,可以看上图3部分。。
> >
> > 2.部署合约  
> > 上图4部分可以看到 Deploy 按钮，输入框输入发行代币的数量，再点击 Deploy部署合约。
> >
> > 合约部署成功后，会出现合约的使用界面。Remix 会自动根据合约的内容，产生对应的合约使用界面。
> >
> > Mytoken合约有两个功能：balanceOf\(查询余额\) 和 transfer\(转移代币\)。如图：
> >
> > ![](/assets/选区_004.png)

**step 6. 执行合约**

> 1.查询余额
>
> > 复制刚才发币用的账户，填入balanceOf 后面的输入框，再点击balanceOf按钮，就会出现这个帐户的余额，如图：
> >
> > ![](/assets/选区_005.png)
>
> 2.转账
>
> > a. account 先选择另一个没有用过的账户作为收款帐户，复制地址如：0x14723a09acff6d2a60dcdf7aa4aff308fddc160c。
> >
> > b. 加上 "," 以及要转账的数额，比如 10,那么完整的输入是："0x14723a09acff6d2a60dcdf7aa4aff308fddc160c, 10"  
> > 一起写入在transfer 后面的输入框里。
> >
> > c. 然后再把account 选择到发币的那个账户，这个意思就是从A账户转移10个代币到B账户,点击transfer就可以执行转账操作
> >
> > d. 通过balanceOf对A账户进行查询，就会发现A账户的余额变成了990个。
> >
> > ![](/assets/选区_006.png)
> >
> > ![](/assets/选区_007.png)

**step 7.用Injected Web3环境测试**

> 1.安装MetaMask
>
> > 先将MetaMask网络环境切换到Ropsten.
> >
> > ![](/assets/009.png)
>
> 2.选择测试网路
>
> > 在 Remix界面， Environment 选择 Injected Web3，这是用于浏览器插件的选项。此时，MetaMask控制您要连接的网络。Remix 会自动连结 MetaMask。
> >
> > Account账户已经显示了我自己的钱包地址了。
> >
> > ![](/assets/选区_008.png)
>
> 3.部署合约
>
> > 用刚才同样的方式，这次尝试发行200个代币，按 create 部署合约，就会看到 MetaMask 的弹出交互窗口，显示了交易信息。需要花费Eth！
> >
> > ![](/assets/10.png)

### 



