### G-01 X+= COSTS MORE GAS THAN X=X+Y FOR STATE VARIABLES 

*Number of Instances Identified: 3*

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

```
66: amountSold += amount;
70: receipts[msg.sender].amount += amount;
71: receipts[msg.sender].balance += uint80(msg.value);
```


### G-02 INCREMENTS CAN BE UNCHECKED

*Number of Instances Identified: 3*

In Solidity 0.8+, there's a default overflow check on unsigned integers. It's possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline

Prior to Solidity 0.8.0, arithmetic operations would always wrap in case of under- or overflow leading to widespread use of libraries that introduce additional checks.

Since Solidity 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

To obtain the previous behaviour, an unchecked block can be used

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol

```
65: for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

```
73: for (uint256 x = temp.currentId + 1; x <= newId; x++) {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol

```
66: for (uint24 x = temp.currentId + 1; x <= newId; x++) {
```




### G-03 INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

*Number of Instances Identified: 1*

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.


https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol

```
67: function _revokeRole(bytes32 _role, address _account) internal override {
```


### G-04 USAGE OF UINTS OR INTS SMALLER THAN 32 BYTES INCURS OVERHEAD

*Number of Instances Identified: 12*

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Use a larger size then downcast where needed


https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol

```
65: for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

```
36: uint48 public amountSold = 0;
49: uint48 x = type(uint48).max;
50: uint80 y = type(uint80).max;
51: uint96 z = type(uint96).max;
59: uint48 amount = uint48(_amount);
67: uint48 newId = amount + temp.currentId;
101: uint80 price = uint80(getPrice()) * r.amount;
102: uint80 owed = r.balance - price;
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol

```
58: uint24 amount = uint24(_amount);
64: uint24 newId = amount + temp.currentId;
66: for (uint24 x = temp.currentId + 1; x <= newId; x++) {
```

