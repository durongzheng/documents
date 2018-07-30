# Oraclize使用注意事项
******
## 加密参数
在使用random.org作为随机数数据源的时候，其语句如下所示：  

```   
/*
NOTE:

Firstly you need to rearrange the format of your Random.org api call to put the encrpytion string at the end of it.
       
Secondly, you need to encrypt your api key using the python script as follows (note the escaped quotes and extra closing curly brace):

python.exe eqt.py -e -p 044992e9473b7d90ca54d2886c7addd14a61109af202f1c95e218b0c99eb060c7134c4ae46345d0383ac996185762f04997d6fd6c393c86e4325c469741e64eca9 "\"YOUR-API-KEY-HERE\"}}"
*/

pragma solidity ^0.4.18;

import "github.com/oraclize/ethereum-api/oraclizeAPI.sol";

import "github.com/Arachnid/solidity-stringutils/strings.sol";

contract TestRoll is usingOraclize {
    using strings for *;
    
    string public lastresult;
    
    string public lasturl;
    
    function playerRollDice() public {       
        uint randomQueryID = 1;
        
        string memory queryString1 = "[URL] ['json(https://api.random.org/json-rpc/1/invoke).result.random[\"data\", \"serialNumber\"]', '\\n{\"jsonrpc\":\"2.0\",\"method\":\"generateSignedIntegers\",\"id\":\"";
        
        string memory queryString2 = uint2str(randomQueryID);
        
        string memory queryString3 = "\",\"params\":{\"n\":\"1\",\"min\":1,\"max\":100,\"replacement\":true,\"base\":10,\"apiKey\":${[decrypt] YOUR-ENCRYPTED-API-KEY-HERE}}']";
     
        string memory queryString1_2 = queryString1.toSlice().concat(queryString2.toSlice());

        string memory queryString = queryString1_2.toSlice().concat(queryString3.toSlice());

        oraclize_query("nested", queryString);

        lasturl = queryString;
    }

    /*
    * semi-public function - only oraclize can call
    */
    /*TLSNotary for oraclize call */
    function __callback(bytes32 myid, string result, bytes proof) public {  
        lastresult = result;
    }
    
}  
```  

由于智能合约代码是开源的，所以如果直接使用你的api key访问random.org的话，就暴露了你自己的api key。而参数加密，则是使用oraclize提供的默认公钥，对你的api key进行加密。之所以要用加密之后的参数，就是为了防止在智能合约代码中暴露你自己访问random.org的api key。假如存在恶意代码，它依然可以用你暴露在智能合约中，用oraclize公钥加密过的参数调用oraclize，从而使用你的api key在random.org的申请次数，一旦这个访问次数超出random.org对你的api key的访问限制，则你的应用的正常访问就无法进行了。为了保护不会受到类似攻击，oraclize保留了每个智能合约的加密参数，只有发布的第一个合约可以使用该参数。    

1. 首先要在random.org申请一个api key，现在random.org还没有正式收费，官方网站上讲是9月份会开始收费，目前申请的是一个beta key；  
2. 要在[这里](https://github.com/oraclize/encrypted-queries)去下载oraclize提供的一个加密CLI命令所需的python文件，给它另取一个名字：eqt.py；  
3. 要执行`python.exe eqt.py -e -p 044992e9473b7d90ca54d2886c7addd14a61109af202f1c95e218b0c99eb060c7134c4ae46345d0383ac996185762f04997d6fd6c393c86e4325c469741e64eca9 "\"YOUR-API-KEY-HERE\"}}"`这个语句，需要现在python中安装`cryptography`和`base58`模块；  
4. 由于我的电脑上同时安装了python27和python37，python27默认没有设置pip的目录，要在环境变量中重新进行一下设置。我的设置方式是：  
	1. 将C:\Python27\Scripts添加到系统环境变量path中;  
	2. 在用户环境变量path中删除了python37的scripts目录；  
5. 用管理员身份运行cmd；  
6. 在c:\python27\scripts目录中，运行`pip install cryptography` 和 `pip install base58` 命令；  
7. python27的pip版本太低，顺便运行了pip 升级命令，将pip升级到了最新版本；
8. 然后就可以执行第3步的命令，生成自己的加密参数了；
9. oraclize针对每一个智能合约会保留其加密参数，并且只有第一个发布的合约可以使用该参数，所以针对每一个发布出来的智能合约，都要重新用`python eqt.py -e -p`命令重新生成一次加密参数；  
