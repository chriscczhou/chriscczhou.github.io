---
title: "Web3隐藏痕迹-使用Multicall模糊化交易"
date: 2023-07-19T0:19:30-04:00
categories:
  - blog
tags:
  - experience
---

## Web3隐藏痕迹-使用Multicall模糊化交易
在以太坊上，使用 multicall 合约可以将多个函数调用合并为一个交易，并让这些函数调用在同一个区块中执行，从而减少交易费用和减少区块链上的交易数量。

multicall 合约代码如下，并在测试网部署：
```
contract Multicall {
    struct Call {
        address target;
        bytes callData;
    }

    function aggregate(Call[] memory calls) public returns (uint256 blockNumber, bytes[] memory returnData) {
        blockNumber = block.number;
        returnData = new bytes[](calls.length);
        for (uint256 i = 0; i < calls.length; i++) {
            (bool success, bytes memory ret) = calls[i].target.call(calls[i].callData);
            require(success);
            returnData[i] = ret;
        }
    }

}
```

multicall 合约通过将多个函数调用的参数封装为一个数组，然后将该数组作为参数传递给 multicall 合约的 aggregate 函数，从而实现在一笔交易中执行多个函数调用。

比如我在测试网想要相对隐蔽的调用0x76f68977535C6360d4D38B9C9c0BEa01Fb9553cB合约的approve方法，可以这么调用，在remix中输入`[["0x76f68977535C6360d4D38B9C9c0BEa01Fb9553cB", "0x095ea7b30000000000000000000000005ab4713df76112d9a97708fc47482ef10a990a470000000000000000000000000000000000000000000000000000000000000000"]]`。

![image](https://github.com/chriscczhou/chriscczhou.github.io/assets/108380177/a8b62421-6121-496d-99cd-bf6b5c288412)

交易确认后其展示在etherscan区块链浏览器上有以下2个特点，在一定程度上可能会让敏感操作不易被发现：
 - 交易会被归类为**Internal Transactions**
 - **Input Data**复杂，对于一般普通人来说不友好

![image](https://github.com/chriscczhou/chriscczhou.github.io/assets/108380177/2f71034a-80e9-40d8-9aaf-d81ab024f4c9)

![image](https://github.com/chriscczhou/chriscczhou.github.io/assets/108380177/3fbc51a3-3ff1-40a8-87e4-dd3087078d5c)

经过测试发现个有趣的事，居然在bscscan上都没被归类为**Internal Transactions**，光看区块链浏览器无法发现，这可能会使项目方偷偷做坏事带来了便利。




