```
#include <eosiolib/eosio.hpp>
#include <string>

using namespace eosio;

using std::string;

class record:public eosio::contract
{

  public:
	// @abi table notes i64;
	struct note
	{
		uint64_t id;
		account_name author;
		string text;

		auto primary_key() const { return id; }
	};

	typedef multi_index<N(notes), note> notes;

	using contract::contract;

	// @abi action
	void write(account_name author, string text)
	{
		require_auth(author);
		print("Hello,", name{author});

		notes nt(_self, _self);

		uint64_t noteId;

		//在表中增加记录
		nt.emplace(author, [&](auto &n) {
			n.id = nt.available_primary_key(); //主键自增
			n.author = author;
			n.text = text;
			noteId = n.id;
		});
		print("----noteId = ", noteId);
	}

	void remove(uint64_t id)
	{
		notes nt(_self, _self);
		auto it = nt.find(id); //查询记录
		eosio_assert(it != nt.end(), "the id not found");
		require_auth(it->author);
		nt.erase(it); //删除一条数据
		　
	}
};

EOSIO_ABI(record, (write)(remove));
```



