QA1. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L64
It might be better to change the line to
```
 require(msg.value == amount * price, "WRONG PRICE");
```