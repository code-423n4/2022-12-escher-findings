# [01] WRONG FUNCTION COMMENT

main/src/minters/FixedPrice.sol: [84](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L84)

```diff
-    /// @notice cancel a fixed price sale
     /// @notice the price of the sale
     function getPrice() public view returns (uint256) {
         return sale.price;
     }
```
