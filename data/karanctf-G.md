1. Reading from `amountSold` which is a state vairable again and again can be very expensive in `LPDA.sol` in `buy` function
```solidity
uint48 public amountSold = 0;


L:66: amountSold += amount;

L:83: uint256 totalSale = price * amountSold;
```
2. Check if `_amount` in `LPDA.sol` in `buy` is zero and revert it in the begaining to save on gas.
3. Use unchecked{--i/++i} insted of i++/i-- in multiple places inside loops
