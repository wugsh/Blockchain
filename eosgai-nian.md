## 多索引表

建多索引表是一种为了在`RAM`快速访问的方法，主要用来来缓存状态和数据。多索引表支持创建、读取、更新和删除(`CRUD`) 业务，区块链不行（它只支持创建和读取）。

多索引表提供了快速访问数据存储接口，是一种存储智能合同中使用的数据的实用的方法。在区块链记录交易信息，你应该使用多索引表存储应用程序数据。

使用多索引表，因为他们支持为使用的数据建立多个索引，主索引必须是`uint64_t`类型和唯一的，但其他的索引，可以有重复的，你可以使用多达`16`个，类型可以是`uint64_t, uint128_t, uint256_t, double or long double`。

如果你想你需要使用一个字符串做索引，需要转换成一个整数型，将结果存储在随后索引的字段中。

###1. 创建一个结构
创建一个可以存储在多索引表中的结构，并在要索引的字段上定义`getter`。

创建一个可以存储在多索引表中的结构，并在要索引的字段上定义`getter`。

请记住，这些getter中必须有一个命名为`primary_key()`，如果没有这个，编译器eosiocpp将产生一个错误…`”it can’t find the field to use as the primary key”`即它找不到任何一个字段被作为主键。

如果你想要有一个以上的索引，（最多允许`16`个），然后为你想要索引的任何字段定义一个`getter`，这时这个名称就不那么重要了，因为你会把`gette`r名称传递给`typedef`。

     ///@abi table
     struct mystruct 
     {
    
    uint64_t key; 
    uint64_t secondid;
    std::string  name; 
    std::string  account; 
    
    uint64_t primary_key() const { return key; } // getter for primary key
    uint64_t by_id() const {return secondid; } // getter for additional key
    
     };
**这里还要注意两件事：**

**a. 注释：**

    /// @abi table
编译器需要使用`eosiocpp`来识别要通过`ABI`公开该表并使其在智能合约之外可见。

**b. 结构名称少于12个字符，而且所有的字符都要小写字母。**

### 2.多索引表和定义索引 ###


定义多索引表将使用`mystruct`，告诉它要索引什么，以及如何获取正在索引的数据。主键将自动创建的，所以使用`struct`后，如果我想要一个只有一个主键的多索引表，我可以定义它为：

    typedef eosio::multi_index<N(mystruct), mystruct> datastore;

这定义了多个索引通过表名`N(mystruct)`和结构名`mystruct`。`N(mystruct)`会对结构名编译转换到`uint64_t`，使用`uint64_t`来标识属于多索引表的数据。

若要添加附加索引或辅助索引，则使用`indexed_b`y模板作为参数，因此定义变为:

    typedef eosio::multi_index<N(mystruct), mystruct, indexed_by<N(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>> datastore;

**注意：**

    indexed_by<N(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>

**参数:**

- 字段的名称转换为整数，`N(secondid)`
- 一个用户定义的密钥调用接口，`const_mem_fun<mystruct, uint64_t, &mystruct::by_id>`

来看看有三个索引的情况。

    /// @abi table
      struct mystruct 
      {
	     uint64_t key; 
	     uint64_t secondid;
	     uint64_t			anotherid;
	     std::string  name; 
	     std::string  account; 
	    
	     uint64_t primary_key() const { return key; }
	     uint64_t by_id() const {return secondid; }
	     uint64_t by_anotherid() const {return anotherid; }
      };
      
    typedef eosio::multi_index<N(mystruct), mystruct, indexed_by<N(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>, indexed_by<N(anotherid), const_mem_fun<mystruct, uint64_t, &mystruct::by_anotherid>>> datastore;
    
这里要注意的一个重要事项是，结构名与表名的匹配，并且将出现在ABI文件中的名称遵循规则（12个字符，所有都是小写的字母）。如果它们没有遵循这个规则，则表不会通过ABI可见（当然可以通过编辑ABI文件来绕过这一点）。

###3.创建定义类型的局部变量
    // local instances of the multi indexes
       pollstable _polls;
       votes _votes;
现在已经定义了一个带有两个索引的多索引表，可以在智能合约中使用它。

如下是一个智能合约使用两个索引的多索引表的例子。在这里你可以看到如何遍历表，如何在同一合约中使用两个表，我们未来将增加额外的教程，利用多索引表。
    
    #include <eosiolib/eosio.hpp>
    
    using namespace eosio;
    
    class youvote : public contract {
      public:
      	youvote(account_name s):contract(s), _polls(s, s), _votes(s, s)
      	{}
    
      	// public methods exposed via the ABI
      	// on pollsTable
    
      	/// @abi action
      	void version()
      	{
      		print("YouVote version  0.01"); 
    
      	};
      
      	/// @abi action
      	void addpoll(account_name s, std::string pollName)
      	{
      		//require_auth(s);
    		print("Add poll ", pollName); 
      	
      		// update the table to include a new poll
      		_polls.emplace(get_self(), [&](auto& p)
      		{
			    p.key = _polls.available_primary_key();
			    p.pollId = _polls.available_primary_key();
			    p.pollName = pollName;
			    p.pollStatus = 0;
			    p.option = "";
			    p.count = 0;
      		});
      	};
    
    
      	/// @abi action
      	void rmpoll(account_name s, std::string pollName)
      	{
      		//require_auth(s);
    	
      		print("Remove poll ", pollName); 
      
	      	std::vector<uint64_t> keysForDeletion;
	      	// find items which are for the named poll
	      	for(auto& item : _polls)
      		{
      			if (item.pollName == pollName)
      			{
      				keysForDeletion.push_back(item.key);   
      			}
      		}
      
      		// now delete each item for that poll
      		for (uint64_t key : keysForDeletion)
      		{
      			print("remove from _polls ", key);
      			auto itr = _polls.find(key);
      			if (itr != _polls.end())
      			{
    				_polls.erase(itr);
     			 }
     		 }
    
    
      // add remove votes ... don't need it the actions are permanently stored on the block chain
    
      std::vector<uint64_t> keysForDeletionFromVotes;
      // find items which are for the named poll
      for(auto& item : _votes)
      {
      	if (item.pollName == pollName)
      	{
      		keysForDeletionFromVotes.push_back(item.key);   
      	}
      }
      
      // now delete each item for that poll
      for (uint64_t key : keysForDeletionFromVotes)
      {
      	print("remove from _votes ", key);
      	auto itr = _votes.find(key);
      	if (itr != _votes.end())
      	{
    		_votes.erase(itr);
      	}
      	}
    
      };
    
      /// @abi action
      void status(std::string pollName)
      {
      	print("Change poll status ", pollName);
    
      	std::vector<uint64_t> keysForModify;
      	// find items which are for the named poll
      	for(auto& item : _polls)
      	{
      		if (item.pollName == pollName)
      	{
      		keysForModify.push_back(item.key);   
      	}
      	}
      
      // now get each item and modify the status
      for (uint64_t key : keysForModify)
      {
    
    	print("modify _polls status", key);
    	auto itr = _polls.find(key);
    	if (itr != _polls.end())
    	{
      		_polls.modify(itr, get_self(), [&](auto& p)
      		{
    			p.pollStatus = p.pollStatus + 1;
      		});
    	}
      }
      };
    
      /// @abi action
      void statusreset(std::string pollName)
      {
      print("Reset poll status ", pollName); 
      
      std::vector<uint64_t> keysForModify;
      // find all poll items
      for(auto& item : _polls)
      {
      if (item.pollName == pollName)
      {
      keysForModify.push_back(item.key);   
      }
      }
      
      // update the status in each poll item
      for (uint64_t key : keysForModify)
      {
      print("modify _polls status", key);
      auto itr = _polls.find(key);
      if (itr != _polls.end())
      {
    _polls.modify(itr, get_self(), [&](auto& p)
    {
      p.pollStatus = 0;
    });
      }
      }
      };
    
    
      /// @abi action
      void addpollopt(std::string pollName, std::string option)
      {
      print("Add poll option ", pollName, "option ", option); 
    
      // find the pollId, from _polls, use this to update the _polls with a new option
      for(auto& item : _polls)
      {
      if (item.pollName == pollName)
      {
    // can only add if the poll is not started or finished
    if(item.pollStatus == 0)
    {
    _polls.emplace(get_self(), [&](auto& p)
      {
    p.key = _polls.available_primary_key();
    p.pollId = item.pollId;
    p.pollName = item.pollName;
    p.pollStatus = 0;
    p.option = option;
    p.count = 0;
      });
    }
    else
    {
    print("Can not add poll option ", pollName, "option ", option, " Poll has started or is finished.");
    }
    
    break; // so you only add it once
      }
      }
      };
    
      /// @abi action
      void rmpollopt(std::string pollName, std::string option)
      {
      print("Remove poll option ", pollName, "option ", option); 
      
      std::vector<uint64_t> keysForDeletion;
      // find and remove the named poll
      for(auto& item : _polls)
      {
      if (item.pollName == pollName)
      {
      keysForDeletion.push_back(item.key);   
      }
      }
      
      
      for (uint64_t key : keysForDeletion)
      {
      print("remove from _polls ", key);
      auto itr = _polls.find(key);
      if (itr != _polls.end())
      {
      if (itr->option == option)
      {
      _polls.erase(itr);
      }
      }
      }
      };
    
    
      /// @abi action
      void vote(std::string pollName, std::string option, std::string accountName)
      {
      print("vote for ", option, " in poll ", pollName, " by ", accountName); 
    
      // is the poll open
      for(auto& item : _polls)
      {
      if (item.pollName == pollName)
      {
      if (item.pollStatus != 1)
      {
      print("Poll ",pollName,  " is not open");
      return;
      }
    
      break; // only need to check status once
      }
      }
    
      // has account name already voted?  
      for(auto& vote : _votes)
      {
      if (vote.pollName == pollName && vote.account == accountName)
      {
      print(accountName, " has already voted in poll ", pollName);
      //eosio_assert(true, "Already Voted");
      return;
      }
      }
    
      uint64_t pollId =99999; // get the pollId for the _votes table
    
      // find the poll and the option and increment the count
      for(auto& item : _polls)
      {
      if (item.pollName == pollName && item.option == option)
      {
      pollId = item.pollId; // for recording vote in this poll
    
      _polls.modify(item, get_self(), [&](auto& p)
    {
    p.count = p.count + 1;
    });
      }
      }
    
      // record that accountName has voted
      _votes.emplace(get_self(), [&](auto& pv)
      {
    pv.key = _votes.available_primary_key();
    pv.pollId = pollId;
    pv.pollName = pollName;
    pv.account = accountName;
      });
      };
    
      private:
    
    // create the multi index tables to store the data
    
      /// @abi table
      struct poll 
      {
    uint64_t  key; // primary key
    uint64_t  pollId; // second key, non-unique, this table will have dup rows for each poll because of option
    std::string   pollName; // name of poll
    uint8_t  pollStatus =0; // staus where 0 = closed, 1 = open, 2 = finished
    std::string  option; // the item you can vote for
    uint32_tcount =0; // the number of votes for each itme -- this to be pulled out to separte table.
    
    uint64_t primary_key() const { return key; }
    uint64_t by_pollId() const {return pollId; }
      };
      typedef eosio::multi_index<N(poll), poll, indexed_by<N(pollId), const_mem_fun<poll, uint64_t, &poll::by_pollId>>> pollstable;
    
    
      /// @abi table
      struct pollvotes 
      {
     uint64_t key; 
     uint64_t pollId;
     std::string  pollName; // name of poll
     std::string  account; //this account has voted, use this to make sure noone votes > 1
    
     uint64_t primary_key() const { return key; }
     uint64_t by_pollId() const {return pollId; }
      };
      typedef eosio::multi_index<N(pollvotes), pollvotes, indexed_by<N(pollId), const_mem_fun<pollvotes, uint64_t, &pollvotes::by_pollId>>> votes;
    
      // local instances of the multi indexes
      pollstable _polls;
      votes _votes;
    };
    
    EOSIO_ABI( youvote, (version)(addpoll)(rmpoll)(status)(statusreset)(addpollopt)(rmpollopt)(vote))
    

注意EOSIO_ABI调用，它通过ABI公开函数，重要的是**函数名**与**ABI函数名**规则一定要匹配。


## EOS多索引表使用指南 ##
**词汇表**

- `code `:是指已公布智能合约的`account_name`。
- `scope`:`account_name`所涉及数据范围。
- `table_name`: 存储在内存中的表的名称。


        #include <eosiolib/eosio.hpp>
    	#include <eosiolib/dispatcher.hpp>
    	#include <eosiolib/multi_index.hpp>
    
    	using namespace eosio;
    
    	namespace limit_order_table {
    
        struct limit_order {
	    uint64_t id;
	    uint128_tprice;
	    uint64_t expiration;
	    account_name owner;
	    
	    auto primary_key() const { return id; }
	    uint64_t get_expiration() const { return expiration; }
	    uint128_t get_price() const { return price; }
	    
	    EOSLIB_SERIALIZE( limit_order, ( id )( price )( expiration )( owner ) )
    	};
    
    	class limit_order_table {
   		public:
    
	    ACTION( N( limitorders ), issue_limit_order ) {
	    	EOSLIB_SERIALIZE( issue_limit_order )
	    };
	    
	    static void on( const issue_limit_order& ilm ) {
	    	auto payer = ilm.get_account();
	    
    		print("Creating multi index table 'orders'.\n");
    		eosio::multi_index< N( orders ), limit_order, indexed_by< N( byexp ),   const_mem_fun< limit_order, uint64_t, &limit_order::get_expiration> >,indexed_by< N( byprice ), const_mem_fun< limit_order, uint128_t, &limit_order::get_price> >> orders( N( limitorders ), N( limitorders ) );
    
    	orders.emplace( payer, [&]( auto& o ) {
		    o.id = 1;
		    o.expiration = 300;
		    o.owner = N(dan);
    	});
    
    	auto order2 = orders.emplace( payer, [&]( auto& o ) {
		    o.id = 2;
		    o.expiration = 200;
		    o.owner = N(thomas);
		});
    
	    print("Items sorted by primary key:\n");
	    for( const auto& item : orders ) {
	    	print(" ID=", item.id, ", expiration=", item.expiration, ", owner=", name{item.owner}, "\n");
	    }
    
    	auto expidx = orders.get_index<N(byexp)>();
    
    	print("Items sorted by expiration:\n");
    	for( const auto& item : expidx ) {
    		print(" ID=", item.id, ", expiration=", item.expiration, ", owner=", name{item.owner}, "\n");
    	}
    
    	auto pridx = orders.get_index<N(byprice)>();
    
   		print("Items sorted by price:\n");
    	for( const auto& item : pridx ) {
    		print(" ID=", item.id, ", expiration=", item.expiration, ", owner=", name{item.owner}, "\n");
    	}
    
    	print("Modifying expiration of order with ID=2 to 400.\n");
    	orders.modify( order2, payer, [&]( auto& o ) {
    		o.expiration = 400;
    	});
    
    	auto lower = expidx.lower_bound(100);
    
    	print("First order with an expiration of at least 100 has ID=", lower->id, " and expiration=", lower->get_expiration(), "\n");
       	};
    
    	} /// limit_order_table
    
    	namespace limit_order_table {
	       	extern "C" {
	      	/// The apply method implements the dispatch of events to this contract
	      		void apply( uint64_t code, uint64_t action ) {
		     		require_auth( code );
		     		eosio_assert( eosio::dispatch< limit_order_table, limit_order_table::issue_limit_order >( code, action ), "Could not dispatch" );
		      	}
		      }
	    }

###**代码分解**
**要存储的结构：**

要在多索引表中存储的数据是`limit_order`结构。`primary_key()，get_expiration()，get_price()`函数用于返回表。返回的表将根据调用的函数排序。

    struct limit_order {
      uint64_t id;
      uint128_tprice;
      uint64_t expiration;
      account_name owner;
    
      auto primary_key() const { return id; }
      uint64_t get_expiration() const { return expiration; }
      uint128_t get_price() const { return price; }
    
      EOSLIB_SERIALIZE( limit_order, ( id )( price )( expiration )( owner ) )
    };
**建一个多索引表：**

    auto payer = ilm.get_account();
	...
`payer`是保存帐户的变量，它将账单元素添加到多索引表中，并修改已经在多索引表中的元素。

	...
	eosio::multi_index< N( orders ), limit_order, 
	...
`N(orders)`是多索引表的名称，`limit_order`是要存储在表中的数据。

    ...  
    indexed_by< N( byexp ), const_mem_fun< limit_order, uint64_t, 
    &limit_order::get_expiration> >,
    ...

`ndexed_by< N( byexp ), const_mem_fun< limit_order, uint64_t, &limit_order::get_expiration>>`定义了多索引表的索引方式。`N(byexp)`是这个索引的名称。`const_mem_fun`表示正在查询的数据类型、`limit_order`的变量的类型是`uint64_t`，将使用`get_expiration`函数获取变量。

    ...
      indexed_by< N( byprice ), const_mem_fun< limit_order, uint128_t, &limit_order::get_price>>
    ...

`indexed_by< N( byprice ), const_mem_fun< limit_order, uint128_t, &limit_order::get_price>>`定义了多索引表的索引方式。`N(byprice)`是这个索引的名称。`const_mem_fun`表示正在查询的数据类型、`limit_order`的变量的类型是`uint128_t`，将使用`get_price`函数获取变量。

    orders( N( limitorders ), N( limitorders )
`orders`即是多索引表。

    auto payer = ilm.get_account();
    
    print("Creating multi index table 'orders'.\n");
    eosio::multi_index< N( orders ), limit_order, 
      indexed_by< N( byexp ),   const_mem_fun< limit_order, uint64_t, &limit_order::get_expiration> >,
      indexed_by< N( byprice ), const_mem_fun< limit_order, uint128_t, &limit_order::get_price> >
    > orders( N( limitorders ), N( limitorders ) );

**添加多索引**

下面，将两个limit_order添加到orders表中。请注意，payer是正在修改orders表的“账单”帐户。

    orders.emplace( payer, [&]( auto& o ) {
      o.id = 1;
      o.expiration = 300;
      o.owner = N(dan);
    });
    
    auto order2 = orders.emplace( payer, [&]( auto& o ) {
      o.id = 2;
      o.expiration = 200;
      o.owner = N(thomas);
    });

**按照主键排序**

默认的`orders`表按照主键排序。

    print("Items sorted by primary key:\n");
    for( const auto& item : orders ) {
      print(" ID=", item.id, ", expiration=", item.expiration, ", owner=", name{item.owner}, "\n");
    }

**按第二索引expiration排序**

`orders`表通过`expiration`进行排序并分配给`expidx`。

    auto expidx = orders.get_index<N(byexp)>();
    
    print("Items sorted by expiration:\n");
    for( const auto& item : expidx ) {
      print(" ID=", item.id, ", expiration=", item.expiration, ", owner=", name{item.owner}, "\n");
    }
**按第二索引price排序**

`orders`表通过`price`进行排序并分配给`pridx`。

**修改一个输入值**
下面，`“ID＝2”`的条目被修改。请注意，`payer`是正在修改`orders`表的“账单”帐户。

    print("Modifying expiration of order with ID=2 to 400.\n");
    orders.modify( order2, payer, [&]( auto& o ) {
      o.expiration = 400;
    });

**得到一个最小值**

    auto lower = expidx.lower_bound(100);
    
    print("First order with an expiration of at least 100 has ID=", lower->id, " and expiration=", lower->get_expiration(), "\n");

**删除表**

表不能直接删除，但是，在删除所有行之后，表将自动删除。
    
## 一张图搞清楚EOS怎么工作 ##
- eosiocpp：一种编译器，允许你将C++编译为可以上传到区块链的格式。
- cleos ：用于将智能合约上传到区块链并查询区块链的命令行工具。
- keosd ：作为守护进程运行的钱包管理器。cleos工具与它交互以便签署请求（需要这样才能信任你给区块链的请求）。
- nodeos：运行区块链本身的服务器软件。

![](https://i.imgur.com/9PhRD2x.jpg)


## EOS的数据存储 ##
使用`eosio::multi_index`接口实现针对EOS智能合约的CRUD操作，该接口带来了可与传统数据库进行比较的功能。

在本文中，我们将创建一个简单的待办事项列表智能合约，用户可以在其中添加待办事项元素`feed the cat`并将其标记为完成。
