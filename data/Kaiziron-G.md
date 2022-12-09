## [G-01] Functions guaranteed to revert when called by normal users can be marked payable

For functions that are guaranteed to revert when called by normal users, such as having `onlyOwner` modifier which should only be called by the owner. Using the payable modifier can lower the gas usage for the intended caller, as it won't include additional checks for the value being sent, which results in fewer opcodes being executed and lower gas usage.

*There are 6 instances of this issue*

```
src/minters/OpenEdition.sol:75:    function cancel() external onlyOwner {
src/minters/FixedPrice.sol:50:    function cancel() external onlyOwner {
src/minters/LPDA.sol:92:    function cancel() external onlyOwner {
src/uris/Base.sol:10:    function setBaseURI(string memory _baseURI) external onlyOwner {
src/uris/Generative.sol:14:    function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {
src/uris/Generative.sol:19:    function setSeedBase() external onlyOwner {
```
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L75
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L50
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L92
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L10
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L14
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L19
