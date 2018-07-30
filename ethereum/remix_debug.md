# 用remix调试智能合约——安装remix-ide

用remix开发以太坊上的智能合约，可以直接使用线上的remix方式：[remix线上版本](https://remix.ethereum.org)  
但是在对智能合约进行调试时，线上版本的VM无法发布合约（至少我的合约发布不上去）。为此，需要安装离线的remix-ide。  
由于在直接使用`npm install remix-ide -g`在我的电脑上报错，安装不起，在此记录下成功的安装过程：  
报错信息是这样的:  
```
C:\Users\durongzheng\AppData\Roaming\npm\node_modules\remix-ide\node_modules\_websocket@1.0.26@websocket>node "C:\Users\durongzheng\AppData\Roaming\npm\node_modules\cnpm\node_modules\npminstall\node-gyp-bin\\node-gyp.js" rebuild
在此解决方案中一次生成一个项目。若要启用并行生成，请添加“/m”开关。  

C:\Users\durongzheng\AppData\Roaming\npm\node_modules\remix-ide\node_modules\_websocket@1.0.26@websocket\build\bufferut
il.vcxproj(20,3): error MSB4019: 未找到导入的项目“C:\Microsoft.Cpp.Default.props”。请确认 <Import> 声明中的路径正确， 且磁盘上存在该文件。

C:\Users\durongzheng\AppData\Roaming\npm\node_modules\remix-ide\node_modules\_websocket@1.0.26@websocket\build\validati
on.vcxproj(20,3): error MSB4019: 未找到导入的项目“C:\Microsoft.Cpp.Default.props”。请确认 <Import> 声明中的路径正确， 且磁盘上存在该文件。

[1/3] scripts.install remixd@0.1.8-alpha.4 › websocket@^1.0.24 finished in 4s

[2/3] scripts.install remixd@0.1.8-alpha.4 › web3@1.0.0-beta.27 › web3-utils@1.0.0-beta.27 › eth-lib@0.1.27 › keccakjs@0.2.1 › sha3@^1.1.0 run "node-gyp rebuild"

```

这个问题是由于node-gyp没有能够正确安装导致的，解决办法如下：  
1. 用管理员权限运行cmd命令：  
2. 引入一个cnpm命令，指向淘宝npm镜像，提高安装速度：
> npm install -g cnpm --registry=https://registry.npm.taobao.org  
> cnpm install -g --production windows-build-tools  
> cnpm install -g remix-ide  

安装成功！
