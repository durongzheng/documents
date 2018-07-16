### 事件Events
  可以使用EVM日志记录工具来创建事件(Events)，在dapp的用户界面中，事件可以用来调用JavaScript的回调函数，回调函数会侦听事件并进行处理。
  事件是合约的可继续成员变量。当被调用时，事件将其参数存入交易日志 - 区块链上一种特殊的数据结构。这些日志是与合约地址关联并合并进了区块链，
  将永久保存在区块链上（【注】原文没有使用永久保存的字眼，而是说只要区块链还可以访问就将存在）。在合约内部是无法访问日志和事件数据的，即便
  是创建日志和事件的合约本身。
  
  如果一个外部调用为合约提供了证明，它可以检查该日志在区块链上是否存在。
  SPV proofs for logs are possible, so if an external entity supplies a contract with such a proof, it can check that the log 
  actually exists inside the blockchain. 
  But be aware that block headers have to be supplied because the contract can only see the last 256 block hashes.
