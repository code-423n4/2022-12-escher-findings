## 1. Non-library/interface files should use fixed compiler versions, not floating ones


    pragma solidity ^0.8.17;

**There are 16 instances of this issue.**

## 2. The struct can be initialized inside the constructor without calling the contract name

        sale = LPDA.Sale(x, x, i, y, y, y, z, i, z);

This can be coded as follows:

        sale = Sale(x, x, i, y, y, y, z, i, z);

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L53