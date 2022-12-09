# Unchecked increment can be used in for-loop

Newer versions of the Solidity compiler will check for integer overflows and underflows automatically. This provides safety but increases gas costs.

When an unsigned integer is guaranteed to never overflow, the unchecked feature of Solidity can be used to save gas costs.

A common case for this is for-loops using a strictly-less-than comparision in their conditional statement, e.g.:
```solidity
uint256 length = someArray.length;
for (uint256 i; i < length; ++i) {
}
```

In cases like this, the maximum value for length is `2**256 - 1`. Therefore, the maximum value of i is `2**256 - 2` as it will always be strictly less than length.

can be replaced with 
```solidity
for (uint256 i; i < length; ) {
  ...
  unchecked {
     ++i;
  }
}
```

This example can be replaced with the following construction to reduce gas costs:

```solidity
File: OpenEdition.sol
66:         for (uint24 x = temp.currentId + 1; x <= newId; x++) {
67:             nft.mint(msg.sender, x);
68:         }

File: LPDA.sol
73:         for (uint256 x = temp.currentId + 1; x <= newId; x++) {
74:             nft.mint(msg.sender, x);
75:         }

File: FixedPrice.sol
65:         for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
66:             nft.mint(msg.sender, x);
67:         }

```
