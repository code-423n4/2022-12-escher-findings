Add a null check for the amount variable for the buy functions.

In the current state, the buy functions can be successfully executed using `amount = 0` (also in combination with `msg.value = 0`), this will not mint any NFTs to the caller, but it will emit `Buy` events that might lead to confusion (even though the amount will show zero).

Locations:
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L58
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L57
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L57