1 - ```x += y``` costs more gas than ```x = x + y``` for state variables
==

Instances of this issue:

- [src/minters/LPDA.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L66)
- [src/minters/LPDA.sol#L70](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L70)
- [src/minters/LPDA.sol#L71](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L71)