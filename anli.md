# 案例1:  用Remix发行自己的代币

Remix [https://remix.ethereum.org/](https://remix.ethereum.org/) 这是最简单方便的智能合约开发环境，直接在浏览器里编写、调试智能合约。

> **step 1. 开发环境**
>
> 浏览器打开：[https://remix.ethereum.org/](https://remix.ethereum.org/)
>
> **step 2.新建合约**
>
> 点击左上角“+”号创建一份新的合约，只能以.sol为后缀，如图：
>
> > ![](/assets/选区_001.png)
>
> **step 3.  编写代币合约代码**  
> 使用solidity编写合约,这个合约只做两件事：
>
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

> **step 4.使用Remix编译合约**
>
> 点击文档右边的 “Start to compile” 开始编译，旁边没有跳出红色的 ERROR 就代表编译成功。
>
> > ![](/assets/选区_002.png)
>
> **step 5.在本地部署合约**
>
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
>
> **step 6. 执行合约**
>
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
>
> **step 7.用Injected Web3环境测试**
>
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

### 2. Truffle开发以太坊DAPP

**step 1.创建一个项目**

> 1.项目初始化
>
> ```
> $ mkdir dapp
> $ cd dapp
> $ truffle unbox webpack
> ```
>
> 执行如图：
>
> ![](/assets/选区_011.png)
>
> 执行以上命令之后，文件夹内会自动生成开发所需要的目录结构：
>
> > contracts/：Solidity职能合约目录
> >
> > migrations/：部署用到的脚本
> >
> > test/：用于测试应用程序和合约的测试文件目录
> >
> > truffle.js：配置文件
> >
> > app：前端代码目录

**step 2.选择以太坊客户端**

安装Ethereum客户端来支持JSON RPC API的调用。

> thereum客户端的选择有很多，本地开发可以使用：Ganache、Ethereumjs-testrpc、以及truffle自带的Truffle Develop，只需要用其中一种就可以。
>
> 主网部署时推荐使用：Geth.

**step 3.编译和部署合约**

> 1.更改truffle.js 文件配置
>
> > 这里要注意的是：
> >
> > Ganache默认运行在7545端口;
> >
> > Ethereumjs-testrpc 默认运行在8545端口;
> >
> > Truffle Develop 默认运行在9545端口;
> >
> > 根据你选择的不同客户端，修改端口，其他代码不要动。
>
> ![](/assets/选区_012.png)
>
> 2.编译合约
>
> > truffle框架里提供demo代码，我们暂时不用写新的合约，可以直接用demo进行编译和部署。
> >
> > 输入命令：
> >
> > ```
> >  $ truffle compile
> > ```
> >
> > 编译成功的话，项目文件夹里会多一个build文件夹。
>
> 3.部署合约  
> 部署合约之前，请开启之前下载的以太坊客户端：
>
> > Ganache：在终端执行 ganache-cli 命令，以出现测试账号为成功，不要关闭，打开新的终端窗口进行下一步;
> >
> > Ethereumjs-testrpc:在终端执行 testrpc 命令，以出现测试账号为成功，不要关闭，打开新的终端窗口进行下一步;
> >
> > Truffle Develop：在终端执行 truffle develop 命令，以出现测试账号为成功，可直接在此窗口进行下一步。
> >
> > 开启客户端之后再输入命令：
> >
> > ```
> >  $ truffle migrate
> > ```
> >
> > 执行成功，会出现以下界面：  
> > ![](/assets/选区_013.png)

**step 4.测试网页与合约交互**

> 上面的合约部署成功后，我们就可以在服务器中查看效果了。执行:
>
> ```
>   $ npm run dev
> ```
>
> ![](/assets/选区_014.png)
>
> 浏览器打开[http://localhost:8080/](http://localhost:8080/) 可以看到一个demo 页面。
>
> ![](/assets/选区_015.png)
>
> 这个是truffle unbox webpack后默认产生的MetaCoin合约，并生成了一个简单的界面。在这里你可以向其他地址发送MetaCoin。
>
> 发送MetaCoin会调用metamask ,所以先将metamask切换到localhost网络环境
>
> 默认端口环境只有 localhost：8545
>
> 如果你用的以太坊客户端是Ganache 或 Truffle Develop的话，就需要Custom RPC 添加 [http://localhost：7545](http://localhost：7545) 或 [http://localhost：9545](http://localhost：9545) 根据你用的客户端来设置。
>
> ![](/assets/20.png)



