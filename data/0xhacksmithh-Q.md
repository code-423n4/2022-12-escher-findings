## QA Report


| |Issue|Instances|
|-|:-|:-:|
| [LOW-1] | transferOwnership() should always be a 2-step process | 2 |
| [LOW-2] | Instead of magic number try to use constant state variables | 3 |
| [LOW-3] | Floating pragma | 12 |



### [Low-01] transferOwnership() should always be a 2-step process

*Instances (2)*:
```solidity
File:   src/uris/Base.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L19
```
```solidity
File:   src/minters/FixedPrice.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L80
```

### [Low-02] Instead of magic number try to use constant state variables

*Instances (3)*:
```solidity
File:   src/minters/FixedPrice.sol

ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L109
```
```solidity
File:   src/minters/LPDA.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L84
```
```solidity
File:   src/minters/OpenEidition.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L92
```


### [Low-03] Floating pragma
All contracts using floating pragma.

Recommended to lock pragma to recent updated and stable version of solidity