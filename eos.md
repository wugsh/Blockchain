`钱包：`

`	创建钱包： cleos wallet create --to-console`

`	解锁钱包： cleos wallet unlock -n walletname`

``

`密钥对：`

`	创建密钥： 		 cleos wallet create_key `

`	查看创建的密钥： cleos wallet private_keys`

`	`

`查看账户信息：     cleos -u http://127.0.0.1:8000 get account cerneweosedu`

`参看当前出块信息： cleos -u http://127.0.0.1:8000 get info`

`查看块信息：       cleos -u http://127.0.0.1:8000 get info block "块号"`

`查看交易信息：     cleos -u http://127.0.0.1:8000 get info transaction "交易号"`

``

`启动节点：`

`	第一次启动节点：   nodeos --genesis-json $EOS_TOOLS_DIR/genesis.json --max-irreversible-block-age 108000000 --data-dir $EOS_DATA_DIR --config-dir $EOS_TOOLS_DIR --delete-all-blocks`

`	以后每次启动节点： nodeos --max-transaction-time 1000 --max-irreversible-block-age 108000000 --data-dir $EOS_DATA_DIR --config-dir $EOS_TOOLS_DIR  `

`	如果数据被写脏：   nodeos --hard-replay-blockchain --wasm-runtime wavm --max-irreversible-block-age 108000000 --data-dir $EOS_TOOLS_DIR --config-dir $EOS_TOOLS_DIR`

``

`投票生产块：`

`	注册节点；cleos -u http://127.0.0.1:8000 system regproducer cerneweosedu EOS7fFrUUs94conpzvEapDsCRLi7VJdbNkUVsnSaENLYxFWYT5SFd`

`	投票抵押；cleos -u http://127.0.0.1:8000 system delegatebw  cerneweosedu cerneweosedu '25000000.0000 EOS' '25000000.0000 EOS'`

`	节点投票；cleos -u http://127.0.0.1:8000 system voteproducer prods cerneweosedu cerneweosedu`

``

``

`查看出块节点： cleos -u http://127.0.0.1:8000  get schedule`

`列出所有BP:    cleos -u http://127.0.0.1:8000 system listproducers`

``

``

`编译合约：`

`	新版本：eosio-cpp -abigen hello.cpp -o hello.wasm`

`	`

`	老版本：`

`		eosiocpp -g hello.abi  hello.cpp`

`		eosiocpp -o hello.wasm hello.cpp`

``

``

`部署合约： cleos -u http://127.0.0.1:8000 set contract cerneweosedu hello/ -p cerneweosedu`

``

`执行合约： cleos -u http://127.0.0.1:8000 push action cerneweosedu hi '["wgs"]' -p cerneweosedu`

`		   cleos -u http://127.0.0.1:8000 push action cerneweosedu hi '{"user" : "wgs"}' -p cerneweosedu`

``

``





