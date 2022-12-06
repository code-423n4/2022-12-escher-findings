# [G-01] It costs less gas to use globally available variables than assigning them to local variables

Saved gas: 10.

## src/uris/Generative.sol:[L22-L24](https://github.com/code-423n4/2022-12-escher/blob/11ca988e39effe6d316e996c27a876c92e82b4da/src/uris/Generative.sol#L22-L24)

```diff
diff --git a/src/uris/Generative.sol b/src/uris/Generative.sol
index db09737..5a3a043 100644
--- a/src/uris/Generative.sol
+++ b/src/uris/Generative.sol
@@ -19,9 +19,14 @@ contract Generative is Unique {
     function setSeedBase() external onlyOwner {
         require(seedBase == bytes32(0), "SEED BASE SET");
 
-        uint256 time = block.timestamp;
-        uint256 numb = block.number;
-        seedBase = keccak256(abi.encodePacked(numb, blockhash(numb - 1), time, (time % 200) + 1));
+        seedBase = keccak256(
+            abi.encodePacked(
+                block.number,
+                blockhash(block.number - 1),
+                block.timestamp,
+                (block.timestamp % 200) + 1
+            )
+        );
     }
```