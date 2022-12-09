### X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES
File: LPDA.sol [Line 66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L66)
```
        amountSold += amount;
```
File: LPDA.sol [Line 70](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L70)
```
        receipts[msg.sender].amount += amount;
```
File: LPDA.sol [Line 71](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L71)
```
        receipts[msg.sender].balance += uint80(msg.value);
```