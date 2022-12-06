# [G-01] USE DIRECTLY ```block``` VALUES TO SAVE ```10``` GAS

main/src/uris/Generative.sol: [L22-L24](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L22-L24)

```diff
index db09737..f76998e 100644
--- a/src/uris/Generative.sol
+++ b/src/uris/Generative.sol
@@ -19,9 +19,7 @@ contract Generative is Unique {
     function setSeedBase() external onlyOwner {
         require(seedBase == bytes32(0), "SEED BASE SET");
 
-        uint256 time = block.timestamp;
-        uint256 numb = block.number;
-        seedBase = keccak256(abi.encodePacked(numb, blockhash(numb - 1), time, (time % 200) + 1));
+        seedBase = keccak256(abi.encodePacked(block.number, blockhash(block.number - 1), block.timestamp, (block.timestamp % 200) + 1));
     }
```

Saves

```
testSetSeedBase() (gas: -10 (-0.030%)) 
```

# [G-02] UNCHECKING ARITHMETICS OPERATIONS THAT CAN’T UNDERFLOW/OVERFLOW (3 INSTANCES)

main/src/minters/FixedPrice.sol: [65](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65)

```diff
-        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
+        for (uint48 x = sale_.currentId + 1; x <= newId;) {
             nft.mint(msg.sender, x);
+            unchecked { ++x; }
```

Saves



main/src/minters/OpenEdition.sol: [66](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66)

```diff
-        for (uint24 x = temp.currentId + 1; x <= newId; x++) {
+        for (uint24 x = temp.currentId + 1; x <= newId;) {
             nft.mint(msg.sender, x);
+            unchecked { ++x; }
         }
```

main/src/minters/LPDA.sol: [73](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73)

```diff
-        for (uint256 x = temp.currentId + 1; x <= newId; x++) {
+        for (uint256 x = temp.currentId + 1; x <= newId;) {
             nft.mint(msg.sender, x);
+            unchecked { ++x; }
         }
```

Results:
```
testSetSeedBase() (gas: -10 (-0.030%)) 
Overall gas change: -10 (-0.030%)
```