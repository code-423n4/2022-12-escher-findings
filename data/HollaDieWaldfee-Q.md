# Escher Low Risk and Non-Critical Issues
## Summary
| Risk      | Title | File | Instances
| ----------- | ----------- | ----------- | ----------- |
| L-00      | Spamming `Buy` events by buying `0` NFTs | - | 3 |
| L-01      | Do not use `selfdestruct` to transfer ETH | - | 2 |
| L-02      | Use `safeMint` instead of `mint` | - | 3 |
| L-03      | Missing zero address checks | - | 6 |
| L-04      | openzeppelin `Ownable` does not use 2-Step-Process to transfer ownership | - | 7 |
| L-05      | Missing sanity checks when creating Sales | - | 2 |
| L-06      | NFTs with tokenId 0 cannot be sold | - | 3 |
| N-00      | Unlocked pragma | - | 12 |

## [L-00] Spamming `Buy` events by buying `0` NFTs
The `FixedPrice.buy`, `OpenEdition.buy` and `LPDA.buy` functions can be called with `_amount=0` and `msg.value=0`.  

This will cause a `Buy` event to be emitted.  

So a malicious user can create `Buy` events, thereby negatively impacting the operation of any components that listen for these events.  

E.g. a UI might display all `Buy` events. If there are a lot of `Buy` events with zero amount, users must sort through them in the UI.  

You can fix this by adding `require(_amount > 0)` to the buy functions.  

These are the 3 instances:  
[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L57-L74](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L57-L74)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L58-L89](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L58-L89)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L57-L72](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L57-L72)  

## [L-01] Do not use `selfdestruct` to transfer ETH
`OpenEdition._end` and `FixedPrice._end` use `selfdestruct` to transfer ETH to the `saleReceiver` address.  

Transferring ETH via `selfdestruct` does not call any logic that is implemented on the receiver side.  

A receiver might implement the `receive()` function that keeps track of incoming payments. By using `selfdestruct` this logic will not be executed.  

So instead of calling `selfdestruct`, the `saleReceiver` should be required to implement some `payable` function that can then be called by the `Sale`.  

You should also make sure that a failed call to this function does not impact the operation of the `Sale` from the buyer perspective.  

After all the ETH balance is transferred to the `saleReceiver` you can then call `selfdestruct` with any address.  

These are the 2 instances:  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L110](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L110)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L122](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L122)  

## [L-02] Use `safeMint` instead of `mint`
It is best practice to use `safeMint` over `mint`. If the receiving address is a contract, `safeMint` requires that it implements the `onERC721Received` function correctly which makes sure that the contract can handle the ERC721 token it receives.  

Make sure to respect the CEI pattern when using `safeMint` or use a reentrancy-guard, as `safeMint` calls an external function.  

There are 3 instances where `mint` is used:  
[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L66)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L74](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L74)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L67](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L67)  

And the `Escher721` contract must also make `safeMint` accessible to addresses with the `MINTER_ROLE`.  

## [L-03] Missing zero address checks
There are 6 instances of missing zero address checks.  
I will list all of them with a quick description of the impact.  

### `feeRecevier`
Accidentally setting `feeReceiver` for a factory to the zero address will cause a loss of fees for all Sales created with that factory.  
[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L43](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L43)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L47](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L47)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L43](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L43)  

### `tokenUriDelegate`
If the `tokenUriDelegate` is the zero address, the NFTs do not have an URI associated with them.  
[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L58](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L58)  

### `_sale.saleReceiver`
If there the `saleReceiver` is the zero address, all sale revenue will be lost.  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L29](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L29)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L29](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L29)  




## [L-04] openzeppelin `Ownable` does not use 2-Step-Process to transfer ownership
Many of the contracts in scope inherit from `Ownable` or `OwnableUpgradeable`. The problem is that these two contracts do not implement a 2-Step-Process for transferring ownership which increases the risk of accidentally transferring ownership to a wrong address, thereby losing ownership of the contract.  

You should consider using `Ownable2Step` ([https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)) and `Ownable2StepUpgradeable` ([https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol)) instead.  

These are the instances where `Ownable` or `OwnableUpgradeable` are inherited from:  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L10](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L10)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L9](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L9)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L10](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L10)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L9](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L9)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L10](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L10)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L9](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L9)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Base.sol#L7](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Base.sol#L7)  

## [L-05] Missing sanity checks when creating Sales
The functions `FixedPriceFactory.createdFixedSale`, `OpenEditionFactory.createOpenEdition` and `LPDAFactory.createLPDASale` all employ sanity checks on the Sale struct paramter.  

However the checks in `FixedPriceFactory.createdFixedSale` and `OpenEditionFactory.createOpenEdition` are less thorough. E.g. they don't check that the price is greater 0. So there should be more sanity checks added (as well as the zero address checks mentioned above).  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L30-L32](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L30-L32)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L30-L32](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L30-L32)  

## [L-06] NFTs with tokenId 0 cannot be sold
tokenIds for ERC721 tokens start at `0`. However the `buy` functions in the Sale contracts only allow buying tokens starting from id `1`. This should be changed such that tokens with id `0` can be sold as well.  

There are 3 instances of this:  
[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66)  

## [N-00] Unlocked pragma
The files in scope use Solidity version `^0.8.17`.  
Consider using a locked version like `0.8.17` to make sure the files only compile with this specific version.  

There are 12 instances of this.  
[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Base.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Base.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Unique.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Unique.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L2)  

[https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L2)  



