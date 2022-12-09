### Report title
- Should use a `custom error` statement instead of using `revert("<Error message written by string type>")` statement

### Code Snippet that has not been optimized yet
- Below are lines that `revert("<Error message written by string type>")` statement is still used and it has not been optimized yet:
  - https://github.com/masaun/2022-12-escher-main/blob/main/src/Escher.sol#L45
  - https://github.com/masaun/2022-12-escher-main/blob/main/src/Escher.sol#L56
```solidity
revert("SoulBound");
```

### Description
- `Custom error` statement that consists of both `error()` statement and `revert()` statement has been able to use since solidity version 0.8.4
   - Using `custom error` statement have been introduced as a more `gas-efficient` way than using `revert("<Error message written by string type>")` statement.
       https://blog.soliditylang.org/2021/04/21/custom-errors/

### Recommendation
- Recommend to replace `revert("<Error message written by string type>")` statement with using `custom error` statement that consists of both `error()` statement and `revert()` statement to save gas.
  (Reference：https://blog.soliditylang.org/2021/04/21/custom-errors/ )

For example,
- In case of unoptimized-line ( https://github.com/masaun/2022-11-stakehouse/blob/main/contracts/liquid-staking/LiquidStakingManager.sol#L734 ), this should be replaced with `custom error` statement like below:
```solidity
contract Escher is ERC1155, AccessControl {

    error SoulBound();   /// @audit - Define a custom error statement like this.

    〜〜Omission〜〜

    /// @notice SoulBound
    function safeTransferFrom(
        address,
        address,
        uint256,
        uint256,
        bytes memory
    ) public pure override {
        revert SoulBound();  /// @audit - Using the custom error statement defined above like this.
    }

    /// @notice SoulBound
    function safeBatchTransferFrom(
        address,
        address,
        uint256[] memory,
        uint256[] memory,
        bytes memory
    ) public pure override {
        revert SoulBound();  /// @audit - Using the custom error statement defined above like this.
    }
```