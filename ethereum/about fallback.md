 # 关于fallback函数
 ### 官方文档中的描述
一个合约可以有一个未命名的函数。这个函数不能有参数，不能返回任何值。当对合约发起调用，但合约中没有与给定的函数描述符(function identifier)匹配的函数，
或者就没有提供调用参数时，就会执行该函数。

另外，当合约只是收到一笔转账(没有调用参数)时，该函数将被执行。除此之外，为了接收Ether，fallback函数必须被标注为`payable`。
如果没有这个函数存在，合约就不能接收普通的转账。

在那种情况下，通常只有很少的gas用于fallback函数调用(准确地说，2300 gas)，所以尽量保持fallback函数便宜非常重要。请注意，一个调用fallback函数的交易所需的gas大得多，因为每一笔交易会增加额外的21000 gas，或者进行其他操作的话，那么所需的gas更多.

下列操作将消耗更多的gas,从而超出提供给fallback函数的可用值:

* 改变状态
* 创建合约
* 调用一个要消耗大量gas的external函数
* 发送ether

在部署合约之前,请确定你完整地测试了你的fallback函数,保证其执行消耗的gas不会超过2300.

`注解`

虽然fallback函数不能有任何参数,但可以通过call调用中的msg.data来接收输入参数(payload).

`警告`

直接给一个没有定义fallback函数的合约转账(不是通过函数调用,而是使用send或transfer),将会抛出异常,ether被返回(在Solidity v0.4.0之前不一样).所以如果你希望你的合约接收ether,你必须实现一个fallback函数.

`警告`

没有payable fallback函数的合约,可以接收挖矿激励(一笔`coinbase`交易)的ether,或作为一个`selfdestruct`函数的接收地址.

合约无法对上述ether转账作出任何反应,也不能拒绝.这是`EVM`和`Solidity`在设计上的选择。

这也意味着`this.balance`可以比合约中的一些手动计数值要高(比如，在fallback函数中有一个计数器，但它无法统计到上述2种转账,所以`this.balance`会比计数器的值高).

```javascript
pragma solidity ^0.4.0;

contract Test {
    // 所有发送到本合约的消息都会调用本函数
    // (这里没有其他函数).
    // 发送ether到这个合约将会引发异常,
    // 是因为它没有`payable`函数修改器
    
    function() public { x = 1; }
    uint x;
}


// 本合约保留所有发送给它的ether，但无法从合约取回.

contract Sink {
    function() public payable { }
}

contract Caller {
    function callTest(Test test) public {
        test.call(0xabcdef01); // hash 不存在
        // 结果是 test.x 变成 == 1.

        // 下面的代码无法通过编译，但即便有人给本合约发送ether，
        // 交易也会失败，ether被返回.
        //test.send(2 ether);
    }
}
```
