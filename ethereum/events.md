# 事件Events

  可以使用EVM日志记录工具来创建事件(Events)，在dapp的用户界面中，事件可以用来调用JavaScript的回调函数，回调函数会侦听事件并进行处理。
  事件是合约的可继续成员变量。当被调用时，事件将其参数存入交易日志 - 区块链上一种特殊的数据结构。这些日志是与合约地址关联并合并进了区块链，
  将永久保存在区块链上（【注】原文没有使用永久保存的字眼，而是说只要区块链还可以访问就将存在）。在合约内部是无法访问日志和事件数据的，即便
  是创建日志和事件的合约本身。
  
  可以进行日志的SPV证明(【注】SPV证明是个什么东东?)，如果一个外部调用为合约提供了该证明，它可以检查该日志在区块链上是否存在。由于合约只能看到最后256个区块哈希，所以请记住提供区块头信息。
  
  最多3个参数可以接受`indexed`索引属性，该属性使得参数可以被检索: 被索引的参数可以在用户界面中使用特定值进行过滤。
  
  如果数组(包括`string`和`bytes`)被用作`indexed`索引参数，它的 Keccak-256 哈希值将被存入事件标题，而非存入事件数据！
  
  事件签名的哈希值是事件标题之一，除非你声明该事件是 `anonymous` 匿名的。`anonymous` 标志意味着该事件不能用名字来过滤。
  
  所有的 (non-indexed)非索引参数将存入日志的数据部分。
  
> 注解 
> 索引参数本身将不会被存储。你只能检索值，而不能接收到值本身。

```javascript
// 合约代码
pragma solidity ^0.4.0;

contract ClientReceipt {
    event Deposit(
        address indexed _from,
        bytes32 indexed _id,
        uint _value
    );

    function deposit(bytes32 _id) public payable {
        // 由于生成了`Deposit`事件，
        // 任何对本函数的调用(即便是很深的嵌套调用)
        // 都可以在JavaScript API中通过 `Deposit` 过滤器被检测到。
        Deposit(msg.sender, _id, msg.value);
    }
}
```

在JavaScript中应该这样使用：
```javascript
var abi = /* 编译器生成的abi */;
var ClientReceipt = web3.eth.contract(abi);
var clientReceipt = ClientReceipt.at("0x1234...ab67" /* 地址 */);

var event = clientReceipt.Deposit();

// 观察变化
event.watch(function(error, result){
    // result 会包含多种信息
    // 包括赋予 `Deposit` 事件的参数
    if (!error)
        console.log(result);
});

// 或者传一个回调函数来立即启动观察
var event = clientReceipt.Deposit(function(error, result) {
    if (!error)
        console.log(result);
});
```

## 日志的底层接口

也可以通过函数 `log0, log1, log2, log3 和 log4`调用日志机制的底层接口。logi 使用 i+1 个 bytes32 类型的参数，这些参数将被存入日志的数据部分，其余则被存入事件标题。上面的事件调用等同于如下代码：
```javascript
//合约代码
pragma solidity ^0.4.10;

contract C {
    function f() public payable {
        bytes32 _id = 0x420042;
        log3(
            bytes32(msg.value),
            bytes32(0x50cb9fe53daa9737b786ab3646f04d0150dc50ef4e75f59509d83667ad5adb20),
            bytes32(msg.sender),
            _id
        );
    }
}
```

代码中长长的16进制数字等于事件的签名，即 keccak256("Deposit(address,hash256,uint256)")。

## 理解事件的更多资源
* [Javascript documentation](https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events)
* [Example usage of events](https://github.com/ethchange/smart-exchange/blob/master/lib/contracts/SmartExchange.sol)
* [How to access them in js](https://github.com/ethchange/smart-exchange/blob/master/lib/exchange_transactions.js)

## 翻译之后的遗留问题
有一些内容没有理解：
1. SPV证明是什么？
2. 事件标题内容过滤，尤其是过滤indexed参数在js中怎么写？
3. 最后一段代码中，log3的参数顺序和Deposit事件的参数顺序不一样，这是怎么回事?
