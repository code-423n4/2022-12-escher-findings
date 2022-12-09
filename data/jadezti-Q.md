## Non Critical Issues

|        | Issue                                                              | Instances |
| ------ |:------------------------------------------------------------------ |:---------:|
| [NC-1] | Missing checks for `address(0)` related to address state variables |     6     |


### [NC-1] Missing checks for `address(0)` 

*Instances (5)*:

```solidity
File: src/minters/LPDAFactory.sol

47:   feeReceiver = fees;
```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol)

```solidity
File: src/uris/Base.sol

11:    baseURI = _baseURI;
```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol)

```solidity
File: src/minters/FixedPriceFactory.sol

43:   feeReceiver = fees;
```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol)

```solidity
File: src/minters/FixedPrice.sol

80:   _transferOwnership(_sale.saleReceiver);
```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)

```solidity
File: src/minters/OpenEditionFactory.sol

43:     feeReceiver = fees;
```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)

```solidity
File: src/minters/OpenEdition.sol

84:    _transferOwnership(_sale.saleReceiver);
```
[Link to code](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

