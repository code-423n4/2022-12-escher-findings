-  `pragma solidity ^0.8.17;` should be `pragma solidity 0.8.17;` to make use of fixed solidity version

- `getPrice()` returns `uint256` however on `line 101` it is casted to `uint80` which may result in loss of data causing unexpected values to be calculated

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L99-L106