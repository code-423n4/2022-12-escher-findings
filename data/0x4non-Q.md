# LOWS / QA / NC

## LOWS

### [L-0] Missing `address(0)` checks

[Escher721.sol#L64-L69](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L64-L69): Missing address(0) check to `receiver`
[LPDAFactory.sol#L46-L48](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46-L48): Missing address(0) check to `fees`
[OpenEditionFactory.sol#L42-L44](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42-L44): Missing address(0) check to `fees`
[FixedPriceFactory.sol#L42-L44](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42-L44): Missing address(0) check to `fees`
[Escher721Factory.sol#L17-L21](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol#L17-L21): on `_escher`

### [L-1] Unsafe downcasting `uint256` to `uint48`
Downcasting, or converting a larger integer type to a smaller one, can potentially lead to loss of data if the value of the larger integer is too big to fit into the smaller type. This can happen in Solidity when using the uint48 type, which can only hold values up to 281474976710655 (or 2^48-1), but is often used to store values that are initially declared as uint256, which has a much larger range of possible values.

In order to prevent data loss, it is important to carefully check the value of the uint256 variable before downcasting it to uint48. This can be done using an if statement and the require function, which will throw an error if the condition is not met.

This happens on:
- [LPDA.sol#L59](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L59): downcasting `_amount`
- [FixedPrice.sol#L62](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L62): downcasting `_amount`


### [L-2] Royalty Fee cant be 100%

On [Escher721.sol#L64-L69](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L64-L69) it seems that `feeNumerator` can be 100_00 bps, add a threashold if you intent to fix a max royalty percentage.

## Non critical

### [NC-0] Floating pragma
Floating pragma may cause unexpected compilation time behaviour and introduce unintended bugs. Is recommend to use only on interfaces and librearies
Change `pragma solidity ^0.8.17;` to `pragma solidity 0.8.17;` on;

- `src/uris/Base.sol:2:pragma solidity ^0.8.17;`
- `src/uris/Generative.sol:2:pragma solidity ^0.8.17;`
- `src/uris/Unique.sol:2:pragma solidity ^0.8.17;`
- `src/minters/OpenEdition.sol:2:pragma solidity ^0.8.17;`
- `src/minters/FixedPrice.sol:2:pragma solidity ^0.8.17;`
- `src/minters/LPDAFactory.sol:2:pragma solidity ^0.8.17;`
- `src/minters/OpenEditionFactory.sol:2:pragma solidity ^0.8.17;`
- `src/minters/FixedPriceFactory.sol:2:pragma solidity ^0.8.17;`
- `src/minters/LPDA.sol:2:pragma solidity ^0.8.17;`
- `src/Escher.sol:2:pragma solidity ^0.8.17;`
- `src/Escher721Factory.sol:2:pragma solidity ^0.8.17;`
- `src/Escher721.sol:2:pragma solidity ^0.8.17;`



### [Q-0] Missing event emission on critical functions
Event emission is a critical aspect of smart contract development in Solidity, as it allows external contracts and applications to listen for and react to changes in the state of a contract. Failure to emit events on key functions can prevent external contracts and applications from functioning properly, and can also make it difficult to debug and track changes to the contract.

- Function `updateURIDelegate` on [Escher721.sol#L57-L59](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L57-L59) should emit an event
- Function `setDefaultRoyalty` on [Escher721.sol#L64-L69](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L64-L69) should emit an event
- Function `resetDefaultRoyalty` on [Escher721.sol#L72-L74](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L72-L74) should emit an event
- Function `setFeeReceiver` on [OpenEditionFactory.sol#L42-L44](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42-L44) should emit an event
- Function `setFeeReceiver` on [LPDAFactory.sol#L46-L48](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46-L48) should emit an event
- Function `setFeeReceiver` on [FixedPriceFactory.sol#L42](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42) should emit an event


### [Q-1] `struct` should be defined before contract states vars

On [OpenEdition.sol#L11](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L11) you have state variable an then the struct. Change it and first declare the structs.
