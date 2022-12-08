## instead of call() , transfer() is used to withdraw the ether

 Description

The usage of send or transfer limits the amount of gas send in the transaction to 2300. This can be considered a safeguard to avoid reentrancy. However, given the current protections of the contracts, it does seem feasible to use call function without further risk. This implies that the system expects to be used by some contracts that can execute more complex code on the fallback function.

This is a classic Code4rena issue:

-   [https://github.com/code-423n4/2021-04-meebits-findings/issues/2](https://github.com/code-423n4/2021-04-meebits-findings/issues/2)
-   [https://github.com/code-423n4/2021-10-tally-findings/issues/20](https://github.com/code-423n4/2021-10-tally-findings/issues/20)
-   [https://github.com/code-423n4/2022-01-openleverage-findings/issues/75](https://github.com/code-423n4/2022-01-openleverage-findings/issues/75)
-   [https://github.com/code-423n4/2022-07-fractional-findings/issues/504](https://github.com/code-423n4/2022-07-fractional-findings/issues/504)




https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L92
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L85-L86
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L109
```solidity
ISaleFactory(factory).feeReceiver().transfer(fee);
temp.saleReceiver.transfer(totalSale - fee);
```


## use safemint

According to OpenZeppelin's suggestion, you should use `Safemint` instead of` mint` to cast NFT

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L66
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L74
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L67

```solidity
for (uint256 x = temp.currentId + 1; x <= newId; x++) {
	nft.mint(msg.sender, x);
}
```