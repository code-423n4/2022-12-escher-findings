## [G-01] Using > 0 costs more gas than != 0 when used on a uint in a require() statement

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance

```
2022-12-escher/src/minters/LPDA.sol::103 => require(owed > 0, "NOTHING TO REFUND");
2022-12-escher/src/minters/LPDAFactory.sol::35 => require(sale.startPrice > 0, "INVALID START PRICE");
2022-12-escher/src/minters/LPDAFactory.sol::36 => require(sale.dropPerSecond > 0, "INVALID DROP PER SECOND");
```

## [G-02] Use Shift Right/Left instead of Division/Multiplication if possible

A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

```
2022-12-escher/src/minters/LPDA.sol::84 => uint256 fee = totalSale / 20;
```

## [G-03] Using private rather than public for constants, saves gas

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```
2022-12-escher/src/Escher721Factory.sol::12 => Escher public immutable escher;
2022-12-escher/src/Escher721Factory.sol::13 => address public immutable implementation;
2022-12-escher/src/minters/FixedPrice.sol::11 => address public immutable factory;
2022-12-escher/src/minters/FixedPriceFactory.sol::13 => address public immutable implementation;
2022-12-escher/src/minters/LPDA.sol::11 => address public immutable factory;
2022-12-escher/src/minters/LPDAFactory.sol::13 => address public immutable implementation;
2022-12-escher/src/minters/OpenEdition.sol::11 => address public immutable factory;
2022-12-escher/src/minters/OpenEditionFactory.sol::13 => address public immutable implementation;
```

## [G-04] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```
2022-12-escher/src/Escher.sol::21 => function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/Escher.sol::27 => function addCreator(address _account) public onlyRole(CURATOR_ROLE) {
2022-12-escher/src/Escher721.sol::51 => function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {
2022-12-escher/src/Escher721.sol::57 => function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {
2022-12-escher/src/Escher721.sol::72 => function resetDefaultRoyalty() public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/minters/FixedPrice.sol::50 => function cancel() external onlyOwner {
2022-12-escher/src/minters/LPDA.sol::92 => function cancel() external onlyOwner {
2022-12-escher/src/minters/OpenEdition.sol::75 => function cancel() external onlyOwner {
2022-12-escher/src/uris/Base.sol::10 => function setBaseURI(string memory _baseURI) external onlyOwner {
2022-12-escher/src/uris/Generative.sol::14 => function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {
2022-12-escher/src/uris/Generative.sol::19 => function setSeedBase() external onlyOwner {
```

## [G-05] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

```
2022-12-escher/src/minters/LPDA.sol::19 => uint80 startPrice;
2022-12-escher/src/minters/LPDA.sol::20 => uint80 finalPrice;
2022-12-escher/src/minters/LPDA.sol::21 => uint80 dropPerSecond;
2022-12-escher/src/minters/LPDA.sol::31 => uint80 balance;
2022-12-escher/src/minters/LPDA.sol::50 => uint80 y = type(uint80).max;
2022-12-escher/src/minters/LPDA.sol::101 => uint80 price = uint80(getPrice()) * r.amount;
2022-12-escher/src/minters/LPDA.sol::102 => uint80 owed = r.balance - price;
```

## [G-06] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, for example when used in for- and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

```
2022-12-escher/src/minters/FixedPrice.sol::65 => for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
2022-12-escher/src/minters/LPDA.sol::73 => for (uint256 x = temp.currentId + 1; x <= newId; x++) {
2022-12-escher/src/minters/OpenEdition.sol::66 => for (uint24 x = temp.currentId + 1; x <= newId; x++) {
```

## [G-07] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

use <x> = <x> + <y> or <x> = <x> - <y> instead to save gas

```
2022-12-escher/src/minters/LPDA.sol::66 => amountSold += amount;
2022-12-escher/src/minters/LPDA.sol::70 => receipts[msg.sender].amount += amount;
2022-12-escher/src/minters/LPDA.sol::71 => receipts[msg.sender].balance += uint80(msg.value);
```

## [G-08] Use custom errors rather than revert()/require() strings to save gas

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they're hitby avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas

```
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

## [G-09] Public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```
2022-12-escher/src/Escher.sol::27 => function addCreator(address _account) public onlyRole(CURATOR_ROLE) {
2022-12-escher/src/Escher721.sol::57 => function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {
2022-12-escher/src/Escher721.sol::72 => function resetDefaultRoyalty() public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/minters/FixedPrice.sol::78 => function initialize(Sale memory _sale) public initializer {
2022-12-escher/src/minters/FixedPrice.sol::86 => function getPrice() public view returns (uint256) {
2022-12-escher/src/minters/FixedPrice.sol::91 => function startTime() public view returns (uint256) {
2022-12-escher/src/minters/FixedPrice.sol::96 => function editionContract() public view returns (address) {
2022-12-escher/src/minters/FixedPrice.sol::101 => function available() public view returns (uint256) {
2022-12-escher/src/minters/FixedPriceFactory.sol::42 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/minters/LPDA.sol::99 => function refund() public {
2022-12-escher/src/minters/LPDA.sol::110 => function initialize(Sale calldata _sale) public initializer {
2022-12-escher/src/minters/LPDA.sol::128 => function startTime() public view returns (uint256) {
2022-12-escher/src/minters/LPDA.sol::133 => function editionContract() public view returns (address) {
2022-12-escher/src/minters/LPDA.sol::138 => function available() public view returns (uint256) {
2022-12-escher/src/minters/LPDA.sol::142 => function lowestPrice() public view returns (uint256) {
2022-12-escher/src/minters/LPDAFactory.sol::46 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/minters/OpenEdition.sol::82 => function initialize(Sale memory _sale) public initializer {
2022-12-escher/src/minters/OpenEdition.sol::89 => function finalize() public {
2022-12-escher/src/minters/OpenEdition.sol::97 => function getPrice() public view returns (uint256) {
2022-12-escher/src/minters/OpenEdition.sol::102 => function startTime() public view returns (uint256) {
2022-12-escher/src/minters/OpenEdition.sol::107 => function timeLeft() public view returns (uint256) {
2022-12-escher/src/minters/OpenEdition.sol::112 => function editionContract() public view returns (address) {
2022-12-escher/src/minters/OpenEdition.sol::116 => function available() public view returns (uint256) {
2022-12-escher/src/minters/OpenEditionFactory.sol::42 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/uris/Base.sol::18 => function initialize(address _owner) public initializer {
```

## [G-10] Use selfbalance()

Use selfbalance() instead of address(this).balance when getting your contract's balance of ETH to save gas.

```
2022-12-escher/src/minters/FixedPrice.sol::109 => ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
2022-12-escher/src/minters/OpenEdition.sol::92 => ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```

## [G-11] internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```
2022-12-escher/src/uris/Generative.sol::27 => function tokenToSeed(uint256 _tokenId) internal view returns (bytes32) {
```

## [G-12] internal functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

```
2022-12-escher/src/uris/Generative.sol::27 => function tokenToSeed(uint256 _tokenId) internal view returns (bytes32) {
```