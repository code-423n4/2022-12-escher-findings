
## Gas Optimizations


|         | Issue                                                                        | Instances |
| ------- |:---------------------------------------------------------------------------- |:---------:|
| [GAS-1] | Using `unchecked` math operations when no overflow/underflow                    |     9     |


### <a name="GAS-1"></a>[GAS-1] Using `unchecked` math operations when no overflow/underflow

*Instances (9)*:

```solidity
File: src/minters/FixedPrice.sol

65:         for (uint48 x = sale_.currentId + 1; x <= newId; x++) {

```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65)

```solidity
File: https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

73:         for (uint256 x = temp.currentId + 1; x <= newId; x++) {

```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73)


```solidity
File: src/minters/OpenEdition.sol

92:   ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);

108:   return sale.endTime > block.timestamp ? sale.endTime - block.timestamp : 0;
```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

```solidity
File: src/minters/OpenEdition.sol

66:    for (uint24 x = temp.currentId + 1; x <= newId; x++) {

```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66)


```solidity
File: https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol

102:    return sale.finalId - sale.currentId;

109:    ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L109)

```solidity
File: https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

84:       uint256 fee = totalSale / 20;

139:     return sale.finalId - sale.currentId;
```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

