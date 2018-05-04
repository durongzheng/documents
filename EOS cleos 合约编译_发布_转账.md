上一篇 [《EOS cleos 钱包_账户_权限_密钥》](https://github.com/durongzheng/documents/blob/master/EOS%20cleos%20%E9%92%B1%E5%8C%85_%E8%B4%A6%E6%88%B7_%E6%9D%83%E9%99%90_%E5%AF%86%E9%92%A5_%E6%93%8D%E4%BD%9C%E5%91%BD%E4%BB%A4.md)
## 合约编译
----------
在git clone下来的源代码中，EOS的示例合约代码在eosio/eos/contracts目录下。一个合约的源代码，一般而言包括一个*.abi描述文件，一个*.cpp源代码文件，一个*.hpp头文件。<br>
如果eos源码编译完全完成的话，那contracts目录下的合约代码也应该被编译完成。合约编译是从c++源代码，经过wasm编译生成wast文件。不知道其他人的环境怎么样，我在编译过程中，contracts目录下的合约没有被编译，合约目录下没有生成wast文件。<br>
eos很贴心地提供了一个eosiocpp工具，封装了wasm编译，可以完成从cpp源文件到wast文件的编译。<br>
如果eos build是完整完成的，则eosiocpp工具在build/tools目录下。如果想在任意目录下eosiocpp命令可用，则到该目录下执行一次make install：
```c
@ubuntu: ~/ $ cd src/eos/build/tools
*@ubuntu: ~/src/eos/build/tools>sudo make install
```
### 关于头文件引用
EOS合同示例代码还不太完善，在编译合约代码时有些时候会出现头文件找不到的报错。ubuntu下默认引用的头文件在/usr/local/include下，合约代码的头文件引用很多都是#include <eosiolib/eosio.hpp>，如果报找不到头文件的错误的话，就需要手动将eosiolib目录（或者其他需要的头文件）拷贝到/usr/local/include目录下。（不少示例代码的头文件引用目录有问题）
## 发布eosio.bios合约
--------------------
首先需要发布eosio.bios合同，据群里边小伙伴讲，这个合同必须首先发布，否则其他合同发布了也起不到作用。解读eosio.bios合约的源代码，这个合约主要是进行一些系统级的限制。<br>
```c
@ubuntu: ~/ $ cd src/eos/contracts/eosio.bios
@ubuntu: ~/src/eos/contracts/eosio.bios $ eosiocpp –o eosio.bios.wast eosio.bios.cpp
```
用eosio账号的active permission发布这个合同：
```c
@ubuntu: ~/src/eos/contracts/eosio.bios $cleos set contract eosio ./ eosio.bios.wast eosio.bios.abi –p eosio@active
```
## 发布eosio.token合约
编译eosio.token合约：
```c
@ubuntu: ~/ $ cd src/eos/contracts/eosio.token
@ubuntu: ~/src/eos/contracts/eosio.token $ eosiocpp –o eosio.token.wast eosio.token.cpp
```
用前面创建的eosio.token账号的active permission发布这个合约：
```c
@ubuntu: ~/src/eos/contracts/eosio.token $cleos set contract eosio.token ./ eosio.token.wast eosio.token.abi –p eosio.token@active
```
创建名为EOS，最大发行量10亿的新代币：
```c
@ubuntu: ~/src/eos/contracts/eosio.token $ cleos push action eosio.token create '["eosio","1000000000.000 EOS",0,0,0]' -p eosio.token
```
给eosio账户发行5亿个EOS
> 这里要注意到的是，eosio.token合约中的create操作，只是创建了10亿个代币，而并没有将这些代币发行给任何账户，包括eosio.token本身。所以，这时如果用cleos get currency balance eosio.token eosio查询的话，eosio账户下是没有代币的。<br>
```c
@ubuntu: ~/src/eos/contracts/eosio.token $cleos push action eosio.token issue '["first","500000000 EOS","first issue"]' -p eosio.token
```
## 用eosio.token合约进行转账
参见上一篇文档[《EOS cleos 钱包_账户_权限_密钥》](https://github.com/durongzheng/documents/blob/master/EOS%20cleos%20%E9%92%B1%E5%8C%85_%E8%B4%A6%E6%88%B7_%E6%9D%83%E9%99%90_%E5%AF%86%E9%92%A5_%E6%93%8D%E4%BD%9C%E5%91%BD%E4%BB%A4.md)<br>
在进行转账之前，先将first,second账户的密钥导入进钱包，并打开钱包、确认钱包已解锁：
```c
@ubuntu: ~/src/eos/contracts/eosio.token$ cleos wallet unlock –n default –-password PW5K8VKeMvc1irxsjpkY3GsaNWNDHhidWrNrWjXZe2RibSujN3xNp
@ubuntu: ~/src/eos/contracts/eosio.token$ cleos wallet import –n default 5JCKpi97Euq3UQ2aW4XFtofHMEGnzkAmsbhrZJbdnUXANQR6ACS
@ubuntu: ~/src/eos/contracts/eosio.token$ cleos wallet import –n default 5KPCXXsKT485tZpkmmNZNHfobBCzYufJ8EqFt156Zi5c69kCc8N
```
相互之间的转账操作：
```c
//eosio转给first
@ubuntu: ~/src/eos/contracts/eosio.token $cleos push action eosio.token transfer '["eosio","first","10000 EOS","fee"]' -p eosio
//first转给second
@ubuntu: ~/src/eos/contracts/eosio.token $cleos push action eosio.token transfer '["first","second","1000 EOS","fee"]' -p first
//second转给eosio
@ubuntu: ~/src/eos/contracts/eosio.token $cleos push action eosio.token transfer '["second","eosio","50 EOS","fee"]' -p second
//看一下他们的余额
@ubuntu: ~/src/eos/contracts/eosio.token $cleos get currency balance eosio.token eosio EOS
@ubuntu: ~/src/eos/contracts/eosio.token $cleos get currency balance eosio.token first EOS
@ubuntu: ~/src/eos/contracts/eosio.token $cleos get currency balance eosio.token second EOS
```
## 合约注意事项
* 发布合约时需要注意，eos中每个账户只能对应发布一个合约。这一点从cleos get code命令后边跟的contract名称，实际上是account_name就可以看出来，获取合约代码的时候，其合同名称实际上就是账户名，也就是说一个账户就对应着一个合约。<br>
* 上一篇文档讲过，账户之间可以共享密钥，所以如果想用同一套密钥发布几个合约的话，在主账户（如eosio)下用同样的密钥为每个合约再创建一个分账户(如eosio.token)就可以；
