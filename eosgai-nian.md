## 多索引表

`建多索引表是一种为了在RAM快速访问的方法，主要用来来缓存状态和数据。多索引表支持创建、读取、更新和删除(CRUD) 业务，区块链不行（它只支持创建和读取）。`

`多索引表提供了快速访问数据存储接口，是一种存储智能合同中使用的数据的实用的方法。在区块链记录交易信息，你应该使用多索引表存储应用程序数据。`

`使用多索引表，因为他们支持为使用的数据建立多个索引，主索引必须是uint64_t类型和唯一的，但其他的索引，可以有重复的，你可以使用多达16个，类型可以是uint64_t, uint128_t, uint256_t, double or long double。`

`如果你想你需要使用一个字符串做索引，需要转换成一个整数型，将结果存储在随后索引的字段中。`

1. ### 创建一个结构

`创建一个可以存储在多索引表中的结构，并在要索引的字段上定义getter。`

`请记住，这些getter中必须有一个命名为primary_key()，如果没有这个，编译器eosiocpp将产生一个错误…”it can’t find the field to use as the primary key”即它找不到任何一个字段被作为主键。`

`如果你想要有一个以上的索引，（最多允许16个），然后为你想要索引的任何字段定义一个getter，这时这个名称就不那么重要了，因为你会把getter名称传递给typedef。`

```cpp
 ///@abi table
 struct mystruct 
 {

    uint64_t     key; 
    uint64_t     secondid;
    std::string  name; 
    std::string  account; 

    uint64_t primary_key() const { return key; } // getter for primary key
    uint64_t by_id() const {return secondid; } // getter for additional key

 };
```

这里还要注意两件事：

1. 注释：

```
/// @abi table
```

  编译器需要使用`eosiocpp`来识别要通过ABI公开该表并使其在智能合约之外可见。

