## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1] | Loops can be more optimizable | 3 |
| [GAS-2] | Functions those could be external instead of public | 22 |
| [GAS-3] | <x> += <y> costs more than <x> = <x> + <y> | 1 |
| [GAS-4] | Repeated require() statement can enclosed inside a modifier then this modifier will further used inside functions | 2 |


### [Gas-01] Loops can be more optimizable
. instead of uint48 use uint256
. instead of x++ try to use ++x
. and uncheck arithmetic operation i.e uncheck{++x}

*Instances (3)*:
```solidity
File:   src/minters/FixedPrice.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65-L67
```
```solidity
File:   src/minters/LPDA.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73-L75
```
```solidity
File:   src/minters/OpenEidition.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66-L68
```

### [Gas-02] Functions those could be external instead of public

*Instances (22)*:
```solidity
File:   src/minters/FixedPrice.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L86
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L91
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L96
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L101
```

```solidity
File:   src/minters/FixedPriceFactory.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42
```
```solidity
File:   src/minters/LPDA.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L128
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L133
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L138
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L142
```
```solidity
File:   src/minters/LPDAFactory.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46
```
```solidity
File:   src/minters/OpenEidition.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L97
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L102
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L107
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L112
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L116
```
```solidity
File:   src/minters/OpenEditionFactory.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42
```
```solidity
File:   src/Escher.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L21
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L27
```
```solidity
File:   src/Escher721.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L57
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L64-L67
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L72
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L78-L80
```

### [Gas-03] <x> += <y> costs more than <x> = <x> + <y>

*Instances (1)*:
```solidity
File:   src/minters/LPDA.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L66
```

### [Gas-04] Repeated require() statement can enclosed inside a modifier then this modifier will further used inside functions
like 
```require(block.timestamp >= temp.startTime, "TOO SOON"); ``` statement used 2 times in ```OpenEidition.sol``` contract file, This can be enclosed inside a modifier and use inside those 2 function.

*Instances (2)*:
```solidity
File:   src/minters/OpenEidition.sol

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L61
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L91

&&

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L62
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L76
```