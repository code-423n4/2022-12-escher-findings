# [Q-01] WRONG FUNCTION COMMENT (7 INSTANCES)

main/src/minters/FixedPrice.sol: [84](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L84)

```diff
-    /// @notice cancel a fixed price sale
     /// @notice the price of the sale
     function getPrice() public view returns (uint256) {
         return sale.price;
     }
```

main/src/minters/OpenEdition.sol: [25](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L25), [55](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L55), [74](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L74).

```diff
-    /// @notice sale info for this fixed price edition
+    /// @notice sale info for this open edition
     Sale public sale;


-    /// @notice buy from a fixed price sale after the sale starts
+    /// @notice buy from a open edition sale after the sale starts
     /// @param _amount the amount of editions to buy
     function buy(uint256 _amount) external payable {

-    /// @notice cancel a fixed price sale
+    /// @notice cancel a open edition sale
     function cancel() external onlyOwner {
```

main/src/minters/LPDA.sol: [34](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L34), [56](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L56), [91](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L91)

```diff
-    /// @notice sale info for this fixed price edition
+    /// @notice sale info for this LPDA edition
     Sale public sale;

-    /// @notice buy from a fixed price sale after the sale starts
+    /// @notice buy from a LPDA sale after the sale starts
     /// @param _amount the amount of editions to buy
     function buy(uint256 _amount) external payable {

-    /// @notice cancel a fixed price sale
+    /// @notice cancel a LPDA sale
     function cancel() external onlyOwner {
```



