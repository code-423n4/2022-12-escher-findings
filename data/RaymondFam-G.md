## Cached Variable Not Fully Utilized
In `OpenEdition.sol`, `buy()` has the struct, `sale`, cached on line 59 as `temp`. However, on line 63, the require statement is using the `state variable` equivalent instead of the stack `local variable`. Consider having the affected code line refactored as follows to save gas as originally intended:

[File: OpenEdition.sol#L63](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L63)   

```
-        require(amount * sale.price == msg.value, "WRONG PRICE");
+        require(amount * temp.price == msg.value, "WRONG PRICE");
```
## Unneeded Cached Variable
In `OpenEdition.sol`, `buy()` has its parameter, `_amount`, cast into uint24 prior to getting it assigned to `amount` on line 58. Caching a function parameter to use it twice has very minimal gas saving benefit unless this pertains to the length of an array getting into a for loop. Consider having the affected code block refactored as follows:

[File: OpenEdition.sol#L57-L72](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L57-L72)

```
58: - uint24 amount = uint24(_amount);
...
64: - uint24 newId = amount + temp.currentId;
    + uint24 newId = uint24(_amount) + temp.currentId; 
...
71: - emit Buy(msg.sender, amount, msg.value, temp);
    + emit Buy(msg.sender, _amount, msg.value, temp);
```
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
[File: FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)

```
86:    function getPrice() public view returns (uint256) {

91:    function startTime() public view returns (uint256) {

96:    function editionContract() public view returns (address) {

101:    function available() public view returns (uint256) {
```
[File: OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

```
97:    function getPrice() public view returns (uint256) {

102:    function startTime() public view returns (uint256) {

107:    function timeLeft() public view returns (uint256) {

112:    function editionContract() public view returns (address) {

116:    function available() public view returns (uint256) {
```
[File: LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

```
117:     function getPrice() public view returns (uint256) {

128:    function startTime() public view returns (uint256) {

133:    function editionContract() public view returns (address) {

138:    function available() public view returns (uint256) {

142:    function lowestPrice() public view returns (uint256) {
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
[File: FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)

```
50:    function cancel() external onlyOwner {
```
[File: OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

```
75:     function cancel() external onlyOwner {
```
[File: LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

```
92:    function cancel() external onlyOwner {
```
## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, consider replacing `>=` with the strict counterpart `>` in the following inequality instance:

[File: FixedPriceFactory.sol#L31](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L31)

```
-        require(_sale.startTime >= block.timestamp, "START TIME IN PAST");
+        require(_sale.startTime > block.timestamp - 1, "START TIME IN PAST");        
```
All other `>=` instances entailed:

[File: OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)

```
31:        require(sale.startTime >= block.timestamp, "START TIME IN PAST");
```
[File: LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol)

```
32:        require(sale.startTime >= block.timestamp, "INVALID START TIME");
```
[File: FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)

```
        require(block.timestamp >= sale_.startTime, "TOO SOON");
```
[File: OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

```
61:        require(block.timestamp >= temp.startTime, "TOO SOON");

91:        require(block.timestamp >= temp.endTime, "TOO SOON");
```
Similarly, consider replacing `<=` with the strict counterpart `<` in the following inequality instance, as an example:

[File: FixedPrice.sol#L63](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L63)

```
-        require(newId <= sale_.finalId, "TOO MANY");
+        require(newId < sale_.finalId + 1, "TOO MANY");
```
All other `<=` instances entailed:

[File: FixedPrice.sol]https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)

```
65:        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
```
[File: OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

```
66:        for (uint24 x = temp.currentId + 1; x <= newId; x++) {
```
## Use Storage Instead of Memory for Structs/Arrays
A storage pointer is cheaper since copying a state struct in memory would incur as many SLOADs and MSTOREs as there are slots. In another words, this causes all fields of the struct/array to be read from storage, incurring a Gcoldsload (2100 gas) for each field of the struct/array, and then further incurring an additional MLOAD rather than a cheap stack read. As such, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables will be much cheaper, involving only Gcoldsload for all associated field reads. Read the whole struct/array into a memory variable only when it is being returned by the function, passed into a function that requires memory, or if the array/struct is being read from another memory array/struct.

Here are the instances entailed:

[File: FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)

```
58:        Sale memory sale_ = sale;
```
[File: OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

```
59:        Sale memory temp = sale;

90:        Sale memory temp = sale
```
[File: LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

```
60:         Sale memory temp = sale;

100:        Receipt memory r = receipts[msg.sender];

118:        Sale memory temp = sale;
```
## += and -= Cost More Gas
`+=` and `-=` generally cost 22 more gas than writing out the assigned equation explicitly. The amount of gas wasted can be quite sizable when repeatedly operated in a loop.

Here are the `+=` instances entailed that may be refactored as follows:

[File: LPDA.sol#L66-L71](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L66-L71)

```
66: - amountSold += amount;
    + amountSold = amountSold + amount;
...
70: - receipts[msg.sender].amount += amount;
    + receipts[msg.sender].amount = receipts[msg.sender].amount + amount;
71: - receipts[msg.sender].balance += uint80(msg.value);
    + receipts[msg.sender].balance = receipts[msg.sender].balance + uint80(msg.value);
```
## Function Order Affects Gas Consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the Optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
  solidity: {
    version: "0.8.17",
    settings: {
      optimizer: {
        enabled: true,
        runs: 1000,
      },
    },
  },
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.