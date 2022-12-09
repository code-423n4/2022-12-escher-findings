# GASOP Report

|         | Issue                                                                                                                              | Instances |
| ------- | :--------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-001] | x += y or x -= y costs more gas than x = x + y or x = x - y for state variables                                                    |     3     |
| [G-002] | `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping` |     6     |
| [G-003] | Multiple accesses of a `mapping/array` should use a local variable cache                                                           |     1     |
| [G-004] | internal functions only called once can be inlined to save gas                                                                     |     2     |

## [G-001] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact

Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings

Total:3

[src/minters/LPDA.sol#L70](https://github.com/code-423n4/2022-12-escher/tree/main//src/minters/LPDA.sol#L70)

```solidity
70:    receipts[msg.sender].amount += amount;
```

[src/minters/LPDA.sol#L71](https://github.com/code-423n4/2022-12-escher/tree/main//src/minters/LPDA.sol#L71)

```solidity
71:    receipts[msg.sender].balance += uint80(msg.value);
```

[src/minters/LPDA.sol#L66](https://github.com/code-423n4/2022-12-escher/tree/main//src/minters/LPDA.sol#L66)

```solidity
66:    amountSold += amount;
```

## [G-002] `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping`

### Impact

It may not be obvious, but every time you copy a storage `struct/array/mapping` to a `memory` variable, you are literally copying each member by reading it from `storage`, which is expensive. And when you use the `storage` keyword, you are just storing a pointer to the storage, which is much cheaper.

### Findings

Total:6

[src/minters/OpenEdition.sol#L59](https://github.com/code-423n4/2022-12-escher/tree/main//src/minters/OpenEdition.sol#L59)

```solidity
59:    Sale memory temp = sale;
```

[src/minters/OpenEdition.sol#L90](https://github.com/code-423n4/2022-12-escher/tree/main//src/minters/OpenEdition.sol#L90)

```solidity
90:    Sale memory temp = sale;
```

[src/minters/LPDA.sol#L60](https://github.com/code-423n4/2022-12-escher/tree/main//src/minters/LPDA.sol#L60)

```solidity
60:    Sale memory temp = sale;
```

[src/minters/LPDA.sol#L118](https://github.com/code-423n4/2022-12-escher/tree/main//src/minters/LPDA.sol#L118)

```solidity
118:    Sale memory temp = sale;
```

[src/minters/LPDA.sol#L100](https://github.com/code-423n4/2022-12-escher/tree/main//src/minters/LPDA.sol#L100)

```solidity
100:    Receipt memory r = receipts[msg.sender];
```

[src/minters/FixedPrice.sol#L58](https://github.com/code-423n4/2022-12-escher/tree/main//src/minters/FixedPrice.sol#L58)

```solidity
58:    Sale memory sale_ = sale;
```

## [G-003] Multiple accesses of a `mapping/array` should use a local variable cache

### Impact

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into `memory/calldata`

### Findings

Total:1

[src/uris/Generative.sol#L16](https://github.com/code-423n4/2022-12-escher/tree/main//src/uris/Generative.sol#L16)

```solidity
16:    scriptPieces[_id] = _data;
```

## [G-004] internal functions only called once can be inlined to save gas

### Impact

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

### Findings

Total:2

[src/Escher.sol#L67](https://github.com/code-423n4/2022-12-escher/tree/main//src/Escher.sol#L67)

```solidity
67:    function _revokeRole(bytes32 _role, address _account) internal override {
```

[src/Escher721.sol#L96](https://github.com/code-423n4/2022-12-escher/tree/main//src/Escher721.sol#L96)

```solidity
96:    function _burn(uint256 tokenId) internal override(ERC721Upgradeable) {
```