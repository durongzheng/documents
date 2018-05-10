教程 智能合约入门 <br/>
==================
Rick Whitner 2018/5/6 编辑了这个页面 · 3修订

智能合约入门
-------------------------------------------
本教程的目的是演示如何设置可用于实验智能合约的本地区块链。本教程的第一部分将着重于讲述：

* 启动私有区块链
* 创建一个钱包
* 加载Bios合约
* 创建帐户
<br>
本教程的第二部分将引导您创建和部署自己的合同。

* eosio.token 合约
* Exchange 合约
* Hello World 合约
<br>
本教程假设您已经安装了EOSIO，并且`nodeos`和`cleos`已经在您的路径中。

启动私有区块链
------------
您可以使用以下命令启动您自己的单节点区块链：
```c/cpp
$ nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::account_history_api_plugin 
...
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
eosio generated block 5e527ee2... #101528 @ 2018-04-01T14:24:58.500 with 0 trxs
```
该命令设置了许多标志并加载了一些可选的插件，我们将在本教程的其余部分中使用这些插件。假设一切正常，你应该每0.5秒看到一次块生成消息。
```c/cpp
eosio generated block 046b9984... #101527 @ 2018-04-01T14:24:58.000 with 0 trxs
```
这意味着您的本地区块链处于活动状态，能生成区块并已可以使用。

有关nodeos您可以使用的参数的更多信息：
```c/cpp
nodeos --help
```
创建一个钱包
-----------
钱包是一个私钥库，是在区块链上执行操作所必需的授权私钥库。这些密钥存储在您的磁盘上，并使用钱包密码进行加密。钱包密码应存储在安全的密码管理器中。
```c/cpp
$ cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JuBXoXJ8JHiCTXfXcYuJabjF9f9UNNqHJjqDVY7igVffe3pXub"
```
为了实现这个简单的开发环境，我们在启动`nodeos`时，启动了`eosio::wallet_api_plugin`插件，您的钱包是您本地的`nodeos`通过该插件进行管理的。任何时候你重新启动nodeos，你必须先解锁你的钱包，然后才能使用其中的密钥。
```c/cpp
$ cleos wallet unlock --password PW5JuBXoXJ8JHiCTXfXcYuJabjF9f9UNNqHJjqDVY7igVffe3pXub
Unlocked: default
```
直接在命令行中使用密码，并将其记录到bash历史记录中通常是不安全的，因此您也可以在交互模式下解锁：
```c/cpp
$ cleos wallet unlock
password:
```
出于安全考虑，通常最好在不使用钱包时锁定钱包。要锁定你的钱包而不关闭`nodeos`，你可以这样做：
```c/cpp
$ cleos wallet lock
Locked: default
```
本教程的其余部分需要您解锁您的钱包。

加载Bios合约
-----------
现在我们有一个带有eosio加载帐户密钥的钱包，我们可以设置默认的系统合同。为了开发的目的，eosio.bios可以使用默认合同。通过此合同，您可以直接控制其他帐户的资源分配并访问其他特权API调用。在公开区块链中，此合同将管理令牌的放样和取消操作，以预留CPU和网络活动的带宽以及合同的内存。

该eosio.bios合同可以在中找到contracts/eosio.bios你的EOSIO源代码文件夹。下面的命令序列假定它是从EOSIO源的根目录执行的，但是您可以通过指定完整路径来从任意位置执行它${EOSIO_SOURCE}/build/contracts/eosio.bios。

$ cleos set contract eosio build/contracts/eosio.bios -p eosio
Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
这个命令序列的结果是cleos用两个动作产生一个事务，eosio::setcode并且eosio::setabi。

代码定义了合约如何运行，abi描述了如何在参数的二进制和json表示之间进行转换。虽然abi在技术上是可选的，但所有的EOSIO工具都依赖于它以便于使用。

任何时候你执行一个事务你都会看到如下输出：

executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
这可以理解为：setcode由定义的动作eosio是通过eosio契约执行的{args...}。

#         ${executor} <= ${contract}:${action} ${args...}
> console output from this execution, if any
正如我们稍后会看到的，行动可以由多个合约处理。

这个电话的最后一个参数是-p eosio。这cleos表示用该eosio账户的主动权限签署此操作，即使用eosio我们先前导入的账户的私钥对操作进行签名。

创建帐户

现在我们已经建立了基本的系统合同，我们可以开始创建自己的账户。我们将创建两个帐户，user而且tester，我们需要将密钥与每个帐户相关联。在这个例子中，两个帐户都使用相同的密钥。

为此，我们首先为账户生成一个密钥。

$ cleos create key
Private key: 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
Public key: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
然后我们将这个密钥导入我们的钱包：

$ cleos wallet import 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
imported private key for: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
注意：确保使用由cleos命令生成的实际键值，而不是上面示例中显示的值！

密钥不会自动添加到钱包，因此跳过此步骤可能会导致您的帐户失去控制权。

创建两个用户帐户

接下来，我们将创建两个帐户，user并tester使用我们上面创建和导入的密钥。

$ cleos create account eosio user EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 8aedb926cc1ca31642ada8daf4350833c95cbe98b869230f44da76d70f6d6242  364 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"user","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...

$ cleos create account eosio tester EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083 366 bytes  1000 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"tester","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...
注意：该create account子命令需要两个密钥，一个用于OwnerKey（在生产环境中应保持高度安全），另一个用于ActiveKey。在本教程示例中，两者都使用相同的密钥。

因为我们正在使用eosio::account_history_api_plugin我们可以查询由我们的密钥控制的所有帐户：

$ cleos get accounts EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
{
  "account_names": [
    "tester",
    "user"
  ]
}
下一步 - Eosio.token，Exchange和Eosio.msig合同

您现在已准备好继续下一个教程，Eosio.token，Exchange和Eosio.msig合同
