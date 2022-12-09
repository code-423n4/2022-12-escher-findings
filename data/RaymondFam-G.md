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
All other instances entailed:

[File: Base.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol)

```
14:    function tokenURI(uint256) external view virtual returns (string memory) {
```
[File: Generative.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol)

```
27:    function tokenToSeed(uint256 _tokenId) internal view returns (bytes32) {
```
[File: Escher.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol)

```
33:    ) public view override(ERC1155, AccessControl) returns (bool) {
```
[File: Escher721.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol)

```
80:    ) public view override(ERC721Upgradeable) returns (string memory) {

90:        returns (bool)
```
## Payable Access Control Functions Costs Less Gas
Consider marking functions with access control as `payable`. This will save 20 gas on each call by their respective permissible callers for not needing to have the compiler check for `msg.value`.

Here are the instances entailed:

[File: Base.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol)

```
10:    function setBaseURI(string memory _baseURI) external onlyOwner {
```
[File: Generative.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol)

```
14:    function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {

19:    function setSeedBase() external onlyOwner {
```
[File: FixedPriceFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol)

```
42:    function setFeeReceiver(address payable fees) public onlyOwner {
```
[File: OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)

```
42:    function setFeeReceiver(address payable fees) public onlyOwner {
```
[File: LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol)

```
46:    function setFeeReceiver(address payable fees) public onlyOwner {
```
[File: Escher.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol)

```
21:    function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {

27:    function addCreator(address _account) public onlyRole(CURATOR_ROLE) {
```
[File: Escher721.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol)

```
51:    function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {

57:    function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {

67:    ) public onlyRole(DEFAULT_ADMIN_ROLE) {

72:    function resetDefaultRoyalty() public onlyRole(DEFAULT_ADMIN_ROLE) {
```
## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, consider replacing `>=` with the strict counterpart `>` in the following inequality instance:

[File: FixedPriceFactory.sol#L31](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L31)

```
-        require(_sale.startTime >= block.timestamp, "START TIME IN PAST");
+        require(_sale.startTime > block.timestamp - 1, "START TIME IN PAST");        
```
All other instances entailed:

[File: OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)

```
31:        require(sale.startTime >= block.timestamp, "START TIME IN PAST");
```
[File: LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol)

```
32:        require(sale.startTime >= block.timestamp, "INVALID START TIME");
```
