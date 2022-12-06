# `Escher._grantRole` has wrong/misleading error message

##### Description

The function `Escher._grantRole` has a [require](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L61) check that prevents addresses from receiving a role that they already have, reverting with `"Already Creator"` in this case. This might be misleading as the function is not only used for `CREATOR_ROLE` but also for `CURATOR_ROLE`.

##### Remediation

Change the error message, e.g.

```diff
diff --git a/src/Escher.sol b/src/Escher.sol
index f666359..29e31f1 100644
--- a/src/Escher.sol
+++ b/src/Escher.sol
@@ -58,7 +58,7 @@ contract Escher is ERC1155, AccessControl {
 
     /// @notice Receive a role, get a token
     function _grantRole(bytes32 _role, address _account) internal override {
-        require(balanceOf(_account, uint256(_role)) == 0, "Already Creator");
+        require(balanceOf(_account, uint256(_role)) == 0, "Already Creator/Curator");
         super._grantRole(_role, _account);
         _mint(_account, uint256(_role), 1, "0x0");
     }
```