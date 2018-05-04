在进行如下操作之前，请确认eos nodeos已经能够正常运行。
## eosio的默认密钥
----------------
Eos testnet eosio账号的默认公、私钥：<br>
公钥："EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV"<br>
私钥："5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3”<br>
## 钱包 wallet
-------------
### 基本概念
* 钱包是保存密钥对的地方，只有钱包中有相应的密钥对(需要导入密钥到钱包中)，并且钱包已经打开并解锁，才能使用该钱包所带账户权限能够进行的操作；
* 钱包的名字可以随便取。`钱包的名字和账户(account)名字没有任何关系`；也就是说，你可以创建一个名为first的账户，同时创建一个名为first的钱包，可first账户的密钥对并不保存在first钱包中，而是保存在default钱包中，只要这两个钱包都是解锁状态，则这样做对后续操作没有任何影响；
* 每次重新连接终端时，`钱包默认是关闭的`，要进行交易型操作，需要先打开钱包并对钱包解锁(cleos wallet open; cleos wallet unlock)；
### 创建钱包
cleos wallet create默认创建一个名为“default”的钱包
```c
@ubuntu:~/eos-wallet$ cleos wallet create
#密码：PW5K8VKeMvc1irxsjpkY3GsaNWNDHhidWrNrWjXZe2RibSujN3xNp
```
记住这个密码。<br>
再创建一个名为first的钱包，同样记住这个密码。
```c
@ubuntu:~/eos-wallet$ cleos wallet create –n first
#密码：PW5HzZacK4SYfPz3nsNJcS3g95w2zyV84FPu65KPYhFz1DaLuQffC
```
打开并解锁钱包，密码就是前边创建钱包时生成的密码，可以在命令行中带上密码选项，也可以不带，在提示输入的时候再输入：
```c
@ubuntu:~/eos-wallet$ cleos wallet open -n default
@ubuntu:~/eos-wallet$ cleos wallet unlock -n default --password PW5K8VKeMvc1irxsjpkY3GsaNWNDHhidWrNrWjXZe2RibSujN3xNp
```
把默认的eosio密钥导入到default钱包：
```c
@ubuntu:~/eos-wallet$ cleos wallet import –n default 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```
这时钱包就可以使用了，可以使用eosio的账户权限进行转账、发布合同等后续操作。<br>
大多数cleos命令行操作都有-p选项，如果使用的是eosio账户，在操作命令行后边加上选项:-p eosio@active。后边的active表明使用的是active permission(权限)。
## 密钥对及账户
-------------
### 基本概念
* 密钥：公钥(public key)、私钥(private key)是区块链应用的基础概念，请参阅相关的资料，网上多得很。私钥一定要离线保存，其安全性至关重要。
* 账户：eos 的账户(account)概念在比特币等区块链应用中是没有的；eos是希望做成分布式的操作系统，所以它要考虑在应用中的权限管理问题，于是引入了账户概念。从我目前研究的情况来看，
