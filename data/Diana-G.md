## G-01 INCREMENTS CAN BE UNCHECKED

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline

Prior to Solidity 0.8.0, arithmetic operations would always wrap in case of under- or overflow leading to widespread use of libraries that introduce additional checks.

Since Solidity 0.8.0, all arithmetic operations revert on over- and underflow by default, thus making the use of these libraries unnecessary.

To obtain the previous behaviour, an unchecked block can be used

_There are 3 instances of this issue_

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol

```
File: src/minters/FixedPrice.sol

65: for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

```
File: src/minters/LPDA.sol

73: for (uint256 x = temp.currentId + 1; x <= newId; x++) {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol

```
File: src/minters/OpenEdition.sol

66: for (uint24 x = temp.currentId + 1; x <= newId; x++) {
```

------

## G-02 x += y COSTS MORE GAS THAN x = x + y FOR STATE VARIABLES (SAME APPLIES FOR x -= y , x = x - y)

_There are 3 instances of this issue_

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

```
File: src/minters/LPDA.sol

66: amountSold += amount;
70: receipts[msg.sender].amount += amount;
71: receipts[msg.sender].balance += uint80(msg.value);
```

-----

## G-03 USAGE OF UINTS OR INTS SMALLER THAN 32 BYTES (26 BITS) INCURS OVERHEAD

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Use a larger size then downcast where needed

_There are 12 instances of this issue_

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol

```
File: src/minters/FixedPrice.sol

65: for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

```
File: src/minters/LPDA.sol

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
File: src/minters/OpenEdition.sol

58: uint24 amount = uint24(_amount);
64: uint24 newId = amount + temp.currentId;
66: for (uint24 x = temp.currentId + 1; x <= newId; x++) {
```

---------

## G-04 INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

_There is 1 instance of this issue_

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol

```
File: src/Escher.sol

67: function _revokeRole(bytes32 _role, address _account) internal override {
```

-----
