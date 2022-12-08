## Use of Named Returns for Local Variables Saves Gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

As an example, the following instance of code block can refactored as follows:

[File: Unique.sol#L10-L11](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol#L10-L11)

```
-    function tokenURI(uint256 _tokenId) external view virtual override returns (string memory) {
-        return string.concat(baseURI, _tokenId.toString());
+    function tokenURI(uint256 _tokenId) external view virtual override returns (string memory _tokenURI) {
+        _tokenURI string.concat(baseURI, _tokenId.toString());
```
[File: Base.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol)

All other instances entailed:

```
14:    function tokenURI(uint256) external view virtual returns (string memory) {
```