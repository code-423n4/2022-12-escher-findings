

### [G01] State variables only set in the constructor should be declared `immutable`

#### Impact
Avoids a Gusset (20000 gas)
#### Findings:
```
2022-12-escher/src/Escher721.sol::22 => address public tokenUriDelegate;
2022-12-escher/src/minters/FixedPrice.sol::27 => Sale public sale;
2022-12-escher/src/minters/FixedPriceFactory.sol::12 => address payable public feeReceiver;
2022-12-escher/src/minters/LPDA.sol::35 => Sale public sale;
2022-12-escher/src/minters/LPDAFactory.sol::12 => address payable public feeReceiver;
2022-12-escher/src/minters/OpenEdition.sol::26 => Sale public sale;
2022-12-escher/src/minters/OpenEditionFactory.sol::12 => address payable public feeReceiver;
2022-12-escher/src/uris/Base.sol::8 => string public baseURI;
2022-12-escher/src/uris/Generative.sol::7 => bytes32 public seedBase;
```



### [G02] State variables can be packed into fewer storage slots

#### Impact
If variables occupying the same slot are both written the same 
function or by the constructor avoids a separate Gsset (20000 gas). 
Reads of the variables are also cheaper
#### Findings:
```
2022-12-escher/src/Escher721.sol::22 => address public tokenUriDelegate;
```



### [G03] `++i/i++` should be `unchecked{++i}`/`unchecked{++i}` when it is not possible for them to overflow, as is the case when used in `for` and `while` loops


#### Findings:
```
2022-12-escher/src/minters/FixedPrice.sol::65 => for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
2022-12-escher/src/minters/LPDA.sol::73 => for (uint256 x = temp.currentId + 1; x <= newId; x++) {
2022-12-escher/src/minters/OpenEdition.sol::66 => for (uint24 x = temp.currentId + 1; x <= newId; x++) {
```

### [G04] Using `> 0` costs more gas than `!= 0` when used on a `uint` in a `require()` statement


#### Findings:
```
2022-12-escher/src/minters/LPDA.sol::103 => require(owed > 0, "NOTHING TO REFUND");
2022-12-escher/src/minters/LPDAFactory.sol::35 => require(sale.startPrice > 0, "INVALID START PRICE");
2022-12-escher/src/minters/LPDAFactory.sol::36 => require(sale.dropPerSecond > 0, "INVALID DROP PER SECOND");
```



### [G05] It costs more gas to initialize variables to zero than to let the default of zero be applied


#### Findings:
```
2022-12-escher/src/minters/LPDA.sol::36 => uint48 public amountSold = 0;
```



### [G06] `++i` costs less gas than `i++`, especially when it’s used in forloops (`--i`/`i--` too)


#### Findings:
```
2022-12-escher/src/minters/FixedPrice.sol::65 => for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
2022-12-escher/src/minters/LPDA.sol::73 => for (uint256 x = temp.currentId + 1; x <= newId; x++) {
2022-12-escher/src/minters/OpenEdition.sol::66 => for (uint24 x = temp.currentId + 1; x <= newId; x++) {
```


### [G07] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

#### Impact
> When using elements that are smaller than 32 bytes, your 
contract’s gas usage may be higher. This is because the EVM operates on 
32 bytes at a time. Therefore, if the element is smaller than that, the 
EVM must use more operations in order to reduce the size of the element 
from 32 bytes to the desired size.
> 

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)
Use a larger size then downcast where needed
#### Findings:
```
2022-12-escher/src/Escher721.sol::66 => uint96 feeNumerator
2022-12-escher/src/minters/FixedPrice.sol::16 => uint48 currentId;
2022-12-escher/src/minters/FixedPrice.sol::17 => uint48 finalId;
2022-12-escher/src/minters/FixedPrice.sol::20 => uint96 price;
2022-12-escher/src/minters/FixedPrice.sol::23 => uint96 startTime;
2022-12-escher/src/minters/FixedPrice.sol::46 => sale = Sale(0, 0, address(0), type(uint96).max, payable(0), type(uint96).max);
2022-12-escher/src/minters/FixedPrice.sol::62 => uint48 newId = uint48(_amount) + sale_.currentId;
2022-12-escher/src/minters/FixedPrice.sol::65 => for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
2022-12-escher/src/minters/LPDA.sol::15 => uint48 currentId;
2022-12-escher/src/minters/LPDA.sol::16 => uint48 finalId;
2022-12-escher/src/minters/LPDA.sol::19 => uint80 startPrice;
2022-12-escher/src/minters/LPDA.sol::20 => uint80 finalPrice;
2022-12-escher/src/minters/LPDA.sol::21 => uint80 dropPerSecond;
2022-12-escher/src/minters/LPDA.sol::23 => uint96 endTime;
2022-12-escher/src/minters/LPDA.sol::26 => uint96 startTime;
2022-12-escher/src/minters/LPDA.sol::30 => uint48 amount;
2022-12-escher/src/minters/LPDA.sol::31 => uint80 balance;
2022-12-escher/src/minters/LPDA.sol::36 => uint48 public amountSold = 0;
2022-12-escher/src/minters/LPDA.sol::49 => uint48 x = type(uint48).max;
2022-12-escher/src/minters/LPDA.sol::50 => uint80 y = type(uint80).max;
2022-12-escher/src/minters/LPDA.sol::51 => uint96 z = type(uint96).max;
2022-12-escher/src/minters/LPDA.sol::59 => uint48 amount = uint48(_amount);
2022-12-escher/src/minters/LPDA.sol::67 => uint48 newId = amount + temp.currentId;
2022-12-escher/src/minters/LPDA.sol::71 => receipts[msg.sender].balance += uint80(msg.value);
2022-12-escher/src/minters/LPDA.sol::82 => sale.finalPrice = uint80(price);
2022-12-escher/src/minters/LPDA.sol::101 => uint80 price = uint80(getPrice()) * r.amount;
2022-12-escher/src/minters/LPDA.sol::102 => uint80 owed = r.balance - price;
2022-12-escher/src/minters/LPDA.sol::117 => function getPrice() public view returns (uint256) {
2022-12-escher/src/minters/OpenEdition.sol::15 => uint72 price;
2022-12-escher/src/minters/OpenEdition.sol::16 => uint24 currentId;
2022-12-escher/src/minters/OpenEdition.sol::19 => uint96 startTime;
2022-12-escher/src/minters/OpenEdition.sol::22 => uint96 endTime;
2022-12-escher/src/minters/OpenEdition.sol::46 => type(uint72).max,
2022-12-escher/src/minters/OpenEdition.sol::47 => type(uint24).max,
2022-12-escher/src/minters/OpenEdition.sol::49 => type(uint96).max,
2022-12-escher/src/minters/OpenEdition.sol::51 => type(uint96).max
2022-12-escher/src/minters/OpenEdition.sol::58 => uint24 amount = uint24(_amount);
2022-12-escher/src/minters/OpenEdition.sol::64 => uint24 newId = amount + temp.currentId;
2022-12-escher/src/minters/OpenEdition.sol::66 => for (uint24 x = temp.currentId + 1; x <= newId; x++) {
2022-12-escher/src/minters/OpenEdition.sol::117 => return sale.endTime > block.timestamp ? type(uint24).max : 0;
```



### [G08] Expressions for constant values such as a call to `keccak256()`, should use `immutable` rather than `constant`

#### Impact
See [this](https://github.com/ethereum/solidity/issues/9232) issue for a detailed description of the issue
#### Findings:
```
2022-12-escher/src/Escher.sol::11 => bytes32 public constant CREATOR_ROLE = keccak256("CREATOR_ROLE");
2022-12-escher/src/Escher.sol::13 => bytes32 public constant CURATOR_ROLE = keccak256("CURATOR_ROLE");
2022-12-escher/src/Escher721.sol::17 => bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");
2022-12-escher/src/Escher721.sol::19 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```



### [G09] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function


#### Findings:
```
2022-12-escher/src/Escher.sol::45 => revert("SoulBound");
2022-12-escher/src/minters/FixedPrice.sol::51 => require(block.timestamp < sale.startTime, "TOO LATE");
2022-12-escher/src/minters/LPDA.sol::62 => require(block.timestamp >= temp.startTime, "TOO SOON");
2022-12-escher/src/minters/LPDAFactory.sol::30 => require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
```



### [G10] `require()` or `revert()` statements that check input arguments should be at the top of the function

#### Impact
Checks that involve constants should come before checks that involve state variables
#### Findings:
```
2022-12-escher/src/minters/LPDAFactory.sol::31 => require(sale.saleReceiver != address(0), "INVALID SALE RECEIVER");
```



### [G11] Use custom errors rather than `revert()`/`require()` strings to save deployment gas


#### Findings:
```
2022-12-escher/src/Escher.sol::45 => revert("SoulBound");
2022-12-escher/src/Escher.sol::56 => revert("SoulBound");
2022-12-escher/src/Escher.sol::61 => require(balanceOf(_account, uint256(_role)) == 0, "Already Creator");
2022-12-escher/src/Escher721Factory.sol::32 => require(escher.hasRole(escher.CREATOR_ROLE(), msg.sender), "NOT AUTHORIZED");
2022-12-escher/src/minters/FixedPrice.sol::51 => require(block.timestamp < sale.startTime, "TOO LATE");
2022-12-escher/src/minters/FixedPrice.sol::60 => require(block.timestamp >= sale_.startTime, "TOO SOON");
2022-12-escher/src/minters/FixedPrice.sol::61 => require(_amount * sale_.price == msg.value, "WRONG PRICE");
2022-12-escher/src/minters/FixedPrice.sol::63 => require(newId <= sale_.finalId, "TOO MANY");
2022-12-escher/src/minters/FixedPriceFactory.sol::30 => require(IEscher721(_sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
2022-12-escher/src/minters/FixedPriceFactory.sol::31 => require(_sale.startTime >= block.timestamp, "START TIME IN PAST");
2022-12-escher/src/minters/FixedPriceFactory.sol::32 => require(_sale.finalId > _sale.currentId, "FINAL ID BEFORE CURRENT");
2022-12-escher/src/minters/LPDA.sol::62 => require(block.timestamp >= temp.startTime, "TOO SOON");
2022-12-escher/src/minters/LPDA.sol::64 => require(msg.value >= amount * price, "WRONG PRICE");
2022-12-escher/src/minters/LPDA.sol::68 => require(newId <= temp.finalId, "TOO MANY");
2022-12-escher/src/minters/LPDA.sol::93 => require(block.timestamp < sale.startTime, "TOO LATE");
2022-12-escher/src/minters/LPDA.sol::103 => require(owed > 0, "NOTHING TO REFUND");
2022-12-escher/src/minters/LPDAFactory.sol::30 => require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
2022-12-escher/src/minters/LPDAFactory.sol::31 => require(sale.saleReceiver != address(0), "INVALID SALE RECEIVER");
2022-12-escher/src/minters/LPDAFactory.sol::32 => require(sale.startTime >= block.timestamp, "INVALID START TIME");
2022-12-escher/src/minters/LPDAFactory.sol::33 => require(sale.endTime > sale.startTime, "INVALID END TIME");
2022-12-escher/src/minters/LPDAFactory.sol::34 => require(sale.finalId > sale.currentId, "INVALID FINAL ID");
2022-12-escher/src/minters/LPDAFactory.sol::35 => require(sale.startPrice > 0, "INVALID START PRICE");
2022-12-escher/src/minters/LPDAFactory.sol::36 => require(sale.dropPerSecond > 0, "INVALID DROP PER SECOND");
2022-12-escher/src/minters/OpenEdition.sol::61 => require(block.timestamp >= temp.startTime, "TOO SOON");
2022-12-escher/src/minters/OpenEdition.sol::62 => require(block.timestamp < temp.endTime, "TOO LATE");
2022-12-escher/src/minters/OpenEdition.sol::63 => require(amount * sale.price == msg.value, "WRONG PRICE");
2022-12-escher/src/minters/OpenEdition.sol::76 => require(block.timestamp < sale.startTime, "TOO LATE");
2022-12-escher/src/minters/OpenEdition.sol::91 => require(block.timestamp >= temp.endTime, "TOO SOON");
2022-12-escher/src/minters/OpenEditionFactory.sol::30 => require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
2022-12-escher/src/minters/OpenEditionFactory.sol::31 => require(sale.startTime >= block.timestamp, "START TIME IN PAST");
2022-12-escher/src/minters/OpenEditionFactory.sol::32 => require(sale.endTime > sale.startTime, "END TIME BEFORE START");
2022-12-escher/src/uris/Generative.sol::15 => require(bytes(scriptPieces[_id]).length == 0, "ALREADY SET");
2022-12-escher/src/uris/Generative.sol::20 => require(seedBase == bytes32(0), "SEED BASE SET");
2022-12-escher/src/uris/Generative.sol::28 => require(seedBase != bytes32(0), "SEED BASE NOT SET");
```



### [G12] Functions guaranteed to revert when called by normal users can be marked `payable`

#### Impact
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
#### Findings:
```
2022-12-escher/src/Escher.sol::21 => function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/Escher.sol::27 => function addCreator(address _account) public onlyRole(CURATOR_ROLE) {
2022-12-escher/src/Escher721.sol::51 => function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {
2022-12-escher/src/Escher721.sol::57 => function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {
2022-12-escher/src/Escher721.sol::72 => function resetDefaultRoyalty() public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/minters/FixedPrice.sol::50 => function cancel() external onlyOwner {
2022-12-escher/src/minters/FixedPriceFactory.sol::42 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/minters/LPDA.sol::92 => function cancel() external onlyOwner {
2022-12-escher/src/minters/LPDAFactory.sol::46 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/minters/OpenEdition.sol::75 => function cancel() external onlyOwner {
2022-12-escher/src/minters/OpenEditionFactory.sol::42 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/uris/Base.sol::10 => function setBaseURI(string memory _baseURI) external onlyOwner {
2022-12-escher/src/uris/Generative.sol::14 => function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {
2022-12-escher/src/uris/Generative.sol::19 => function setSeedBase() external onlyOwner {
```




### [G13] Use `calldata` instead of `memory` for function parameters

#### Impact
Use calldata instead of memory for function parameters. Having function arguments use calldata instead of memory can save gas.
#### Findings:
```
2022-12-escher/src/Escher.sol::21 => function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/minters/FixedPrice.sol::78 => function initialize(Sale memory _sale) public initializer {
2022-12-escher/src/minters/FixedPrice.sol::107 => function _end(Sale memory _sale) internal {
2022-12-escher/src/minters/OpenEdition.sol::82 => function initialize(Sale memory _sale) public initializer {
2022-12-escher/src/minters/OpenEdition.sol::120 => function _end(Sale memory _sale) internal {
2022-12-escher/src/uris/Base.sol::10 => function setBaseURI(string memory _baseURI) external onlyOwner {
2022-12-escher/src/uris/Generative.sol::14 => function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {
```


### [G14] Amounts should be checked for 0 before calling a transfer

#### Impact
Checking non-zero transfer values can avoid an expensive external call and save gas.

While this is done in some places, it’s not consistently done in the solution.
#### Findings:
```
2022-12-escher/src/minters/FixedPrice.sol::109 => ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
2022-12-escher/src/minters/LPDA.sol::85 => ISaleFactory(factory).feeReceiver().transfer(fee);
2022-12-escher/src/minters/LPDA.sol::86 => temp.saleReceiver.transfer(totalSale - fee);
2022-12-escher/src/minters/LPDA.sol::105 => payable(msg.sender).transfer(owed);
2022-12-escher/src/minters/OpenEdition.sol::92 => ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```




### [G15] Using `private` rather than `public` for constants, saves gas

#### Impact
If needed, the value can be read from the verified contract source 
code.
#### Findings:
```
2022-12-escher/src/Escher.sol::11 => bytes32 public constant CREATOR_ROLE = keccak256("CREATOR_ROLE");
2022-12-escher/src/Escher.sol::13 => bytes32 public constant CURATOR_ROLE = keccak256("CURATOR_ROLE");
2022-12-escher/src/Escher721.sol::17 => bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");
2022-12-escher/src/Escher721.sol::19 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```


### [G16] Empty blocks should be removed or emit something

#### Impact
The code should be refactored such that they no longer exist, or the 
block should do something useful, such as emitting an event or 
reverting. If the contract is meant to be extended, the contract should 
be abstract and the function signatures be added without 
any default implementation. If the block is an empty if-statement block 
to avoid doing subsequent checks in the else-if/else conditions, the 
else-if/else conditions should be nested under the negation of the 
if-statement, because they involve different classes of checks, which 
may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})
#### Findings:
```
2022-12-escher/src/Escher721.sol::25 => constructor() {}
```

