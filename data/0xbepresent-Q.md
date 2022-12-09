1 - _safeMint() should be used rather than _mint()
==

```_mint()``` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol) in favor of ```_safeMint()``` which ensures that the recipient is either an EOA or contract which implements IERC721Receiver.

Instance of this issue:

- [minters/OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol)
- [src/minters/FixedPrice.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L66)
- [src/minters/LPDA.sol#L74](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L74)

2 - Lock pragma version
==

Floating pragmas should not be used. Locking the pragma helps to ensure that contracts don't accidentally get deployed using outdated compiler version that might introduce bugs.

change ```pragma solidity ^0.8.17;``` to ```pragma solidity 0.8.17```;

3 - Validates price is non-zero
==

In the ```FixedPriceFactory.sol::createFixedSale()``` and ```OpenEditionFactory.sol::createOpenEdition()``` add a non-zero price validation.

Links:

- [src/minters/FixedPriceFactory.sol#L29](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L29)
- [src/minters/OpenEditionFactory.sol#L29](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L29)