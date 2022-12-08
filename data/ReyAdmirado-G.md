| | issue |
| ----------- | ----------- |
| 1 | [instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant](#1-instead-of-calculating-a-statevar-with-keccak256-every-time-the-contract-is-made-pre-calculate-them-before-and-only-give-the-result-to-a-constant) |
| 2 | [expressions for constant values such as a call to keccak256(), should use immutable rather than constant](#2-expressions-for-constant-values-such-as-a-call-to-keccak256-should-use-immutable-rather-than-constant) |
| 3 | [make the stack var when its needed instead of making it early](#3-make-the-stack-var-when-its-needed-instead-of-making-it-early) |
| 4 | [dont use state var here instead use stack var or argument](#4-dont-use-state-var-here-instead-use-stack-var-or-argument) |
| 5 | [Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement](#5-add-unchecked--for-subtractions-where-the-operands-cannot-underflow-because-of-a-previous-require-or-if-statement) |
| 6 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#6-x--y-costs-more-gas-than-x--x--y-for-state-variables) |
| 7 | [`++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops](#7-ii-should-be-uncheckediuncheckedi-when-it-is-not-possible-for-them-to-overflow-as-is-the-case-when-used-in-for-loop-and-while-loops) |
| 8 | [require() or revert() statements that check input arguments should be at the top of the function](#8-require-or-revert-statements-that-check-input-arguments-should-be-at-the-top-of-the-function) |
| 9 | [bytes constants are more efficient than string constants](#9-bytes-constants-are-more-efficient-than-string-constants) |
| 10 | [public functions not called by the contract should be declared external instead](#10-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 11 | [state variables can be packed into fewer storage slots](#11-state-variables-can-be-packed-into-fewer-storage-slots) |




## 1. instead of calculating a statevar with keccak256() every time the contract is made pre calculate them before and only give the result to a constant

- [Escher.sol#L11](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L11)
- [Escher.sol#L13](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L13)

- [Escher721.sol#L17](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L17)
- [Escher721.sol#L19](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L19)


## 2. expressions for constant values such as a call to keccak256(), should use immutable rather than constant

- [Escher.sol#L11](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L11)
- [Escher.sol#L13](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L13)

- [Escher721.sol#L17](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L17)
- [Escher721.sol#L19](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L19)


## 3. make the stack var when its needed instead of making it early

when stack var is not needed in the require checks or if statements make it after them and right before its needed to possibly save gas

make `nft` before the for loop (line 65)
- [FixedPrice.sol#L59](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L59)
- [OpenEdition.sol#L60](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L60)
- [LPDA.sol#L61](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L61)

make `amount` before the third require check
- [OpenEdition.sol#L58](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L58)

make `amount` before the second require check
- [LPDA.sol#L59](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L59)


## 4. dont use state var here instead use stack var or argument

97 gas cheaper

use `temp.price` instead of `sale.price` 
- [OpenEdition.sol#L63](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L63)

use `_sale.saleReceiver` instead of `sale.saleReceiver`
- [LPDA.sol#L112](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L112)
- [OpenEdition.sol#L84](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L84)


## 5. Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
this will stop the check for overflow and underflow so it will save gas

`sale.endTime - block.timestamp` will not underflow
- [OpenEdition.sol#L108](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L108)

although not checked before it `numb - 1` will never underflow so we can use unchecked{} for it 
- [Generative.sol#L24](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L24)

`totalSale` is 20 times bigger than `fee` so no way it underflows
- [LPDA.sol#L86](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L86)

`block.timestamp - start` cant underflow because of check in line 120
- [LPDA.sol#L123](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L123)


## 6. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas

- [LPDA.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L66)
- [LPDA.sol#L70](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L70)
- [LPDA.sol#L71](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L71)


## 7. `++i/i++` should be `unchecked{++i}/unchecked{i++}` when it is not possible for them to overflow, as is the case when used in for-loop and while-loops

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

- [FixedPrice.sol#L65](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65)

- [OpenEdition.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66)

- [LPDA.sol#L73](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73)


## 8. require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

swap the require of L30 to after the other 2 
- [FixedPriceFactory.sol#L30-L32](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L30-L32)

swap the require of L30 to after the other 2
- [OpenEditionFactory.sol#L30-L32](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L30-L32)

swap the require of L30 to after all the others
- [LPDAFactory.sol#L30](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L30)


## 9. bytes constants are more efficient than string constants

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

- [Base.sol#L8](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L8)

- [Generative.sol#L9](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L9)


## 10. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

`setFeeReceiver`
- [FixedPriceFactory.sol#L42](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42)

`setFeeReceiver`
- [OpenEditionFactory.sol#L42](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42)

`getPrice`
- [FixedPrice.sol#L86](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L86)
- [OpenEdition.sol#L97](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L97)
- [LPDA.sol#L129](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L129)

`startTime`
- [FixedPrice.sol#L91](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L91)
- [OpenEdition.sol#L102](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L102)
- [LPDA.sol#L134](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L134)

`editionContract`
- [FixedPrice.sol#L96](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L96)
- [OpenEdition.sol#L107](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L107)
- [LPDA.sol#L134](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L134)

`available`
- [FixedPrice.sol#L101](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L101)
- [OpenEdition.sol#L112](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L112)
- [LPDA.sol#L139](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L139)

`lowestPrice`
- [LPDA.sol#L142](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L142)

`timeLeft`
- [OpenEdition.sol#L116](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L116)

`addCreator`
- [Escher.sol#L27](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L27)

`safeTransferFrom`
- [Escher.sol#L38](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L38)

`safeBatchTransferFrom`
- [Escher.sol#L49](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L49)

`resetDefaultRoyalty`
- [Escher721.sol#L72](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L72)

`updateURIDelegate`
- [Escher721.sol#L57](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L57)

`setFeeReceiver`
- [Escher721.sol#L46](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L46)

`refund`
- [LPDA.sol#L99](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L99)


## 11. state variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

we can put `amountSold` after `factory` to make them into a single slot (factory is in line 11)
- [LPDA.sol#L36](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L36)















