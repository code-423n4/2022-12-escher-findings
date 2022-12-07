## [L-01] Unspecific Compiler Version Pragma

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

```
2022-12-escher/src/Escher.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/Escher721.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/Escher721Factory.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/FixedPrice.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/FixedPriceFactory.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/LPDA.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/LPDAFactory.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/OpenEdition.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/OpenEditionFactory.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/uris/Base.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/uris/Generative.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/uris/Unique.sol::2 => pragma solidity ^0.8.17;
```

## [L-02] Use of Block.timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

```
2022-12-escher/src/minters/FixedPrice.sol::51 => require(block.timestamp < sale.startTime, "TOO LATE");
2022-12-escher/src/minters/FixedPrice.sol::60 => require(block.timestamp >= sale_.startTime, "TOO SOON");
2022-12-escher/src/minters/FixedPriceFactory.sol::31 => require(_sale.startTime >= block.timestamp, "START TIME IN PAST");
2022-12-escher/src/minters/LPDA.sol::62 => require(block.timestamp >= temp.startTime, "TOO SOON");
2022-12-escher/src/minters/LPDA.sol::93 => require(block.timestamp < sale.startTime, "TOO LATE");
2022-12-escher/src/minters/LPDA.sol::120 => if (block.timestamp < start) return type(uint256).max;
2022-12-escher/src/minters/LPDA.sol::123 => uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;
2022-12-escher/src/minters/LPDAFactory.sol::32 => require(sale.startTime >= block.timestamp, "INVALID START TIME");
2022-12-escher/src/minters/OpenEdition.sol::61 => require(block.timestamp >= temp.startTime, "TOO SOON");
2022-12-escher/src/minters/OpenEdition.sol::62 => require(block.timestamp < temp.endTime, "TOO LATE");
2022-12-escher/src/minters/OpenEdition.sol::76 => require(block.timestamp < sale.startTime, "TOO LATE");
2022-12-escher/src/minters/OpenEdition.sol::91 => require(block.timestamp >= temp.endTime, "TOO SOON");
2022-12-escher/src/minters/OpenEdition.sol::108 => return sale.endTime > block.timestamp ? sale.endTime - block.timestamp : 0;
2022-12-escher/src/minters/OpenEdition.sol::117 => return sale.endTime > block.timestamp ? type(uint24).max : 0;
2022-12-escher/src/minters/OpenEditionFactory.sol::31 => require(sale.startTime >= block.timestamp, "START TIME IN PAST");
2022-12-escher/src/uris/Generative.sol::22 => uint256 time = block.timestamp;
```

## [L-03] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). Unless there is a compelling reason, abi.encode should be preferred. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.

```
2022-12-escher/src/uris/Generative.sol::24 => seedBase = keccak256(abi.encodePacked(numb, blockhash(numb - 1), time, (time % 200) + 1));
2022-12-escher/src/uris/Generative.sol::29 => return keccak256(abi.encodePacked(_tokenId, seedBase));
```

## [L-04] Unsafe use of transfer()/transferFrom() with IERC20

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert. Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

```
2022-12-escher/src/minters/FixedPrice.sol::109 => ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
2022-12-escher/src/minters/LPDA.sol::85 => ISaleFactory(factory).feeReceiver().transfer(fee);
2022-12-escher/src/minters/LPDA.sol::86 => temp.saleReceiver.transfer(totalSale - fee);
2022-12-escher/src/minters/OpenEdition.sol::92 => ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```

## [L-05] Events not emitted for important state changes

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

```
2022-12-escher/src/Escher.sol::21 => function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/Escher721.sol::64 => function setDefaultRoyalty(
2022-12-escher/src/minters/FixedPriceFactory.sol::42 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/minters/LPDAFactory.sol::46 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/minters/OpenEditionFactory.sol::42 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/uris/Base.sol::10 => function setBaseURI(string memory _baseURI) external onlyOwner {
2022-12-escher/src/uris/Generative.sol::14 => function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {
2022-12-escher/src/uris/Generative.sol::19 => function setSeedBase() external onlyOwner {
```

## [L-06] _safeMint() should be used rather than _mint() wherever possible

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert. Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

```
2022-12-escher/src/Escher.sol::63 => _mint(_account, uint256(_role), 1, "0x0");
2022-12-escher/src/Escher721.sol::52 => _mint(to, tokenId);
```

## [L-07] Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions

__gap is empty reserved space in storage that is recommended to be put in place in upgradeable contracts. It allows new state variables to be added in the future without compromising the storage compatibility with existing deployments

```
2022-12-escher/src/minters/FixedPrice.sol::10 => contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
2022-12-escher/src/minters/LPDA.sol::10 => contract LPDA is Initializable, OwnableUpgradeable, ISale {
2022-12-escher/src/minters/OpenEdition.sol::10 => contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
2022-12-escher/src/uris/Base.sol::7 => contract Base is ITokenUriDelegate, OwnableUpgradeable {
```

## [L-08] Implementation contract may not be initialized

Implementation contract does not have a constructor with the initializer modifier therefore may be uninitialized. Implementation contracts should be initialized to avoid potential griefs or exploits.

```
2022-12-escher/src/uris/Base.sol::7 => contract Base is ITokenUriDelegate, OwnableUpgradeable {
```

## [N-01] Event is missing indexed fields

Each event should use three indexed fields if there are three or more fields

```
2022-12-escher/src/minters/FixedPrice.sol::31 => event Start(Sale _saleInfo);
2022-12-escher/src/minters/FixedPrice.sol::34 => event End(Sale _saleInfo);
2022-12-escher/src/minters/LPDA.sol::41 => event Start(Sale _saleInfo);
2022-12-escher/src/minters/LPDA.sol::42 => event End(Sale _saleInfo);
2022-12-escher/src/minters/OpenEdition.sol::30 => event Start(Sale _saleInfo);
2022-12-escher/src/minters/OpenEdition.sol::33 => event End(Sale _saleInfo);
```

## [N-02] Missing NatSpec

Code should include NatSpec

```
2022-12-escher/src/uris/Base.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/uris/Unique.sol::1 => // SPDX-License-Identifier: MIT
```

## [N-03] Missing event for critical parameter change

Emitting events after sensitive changes take place, to facilitate tracking and notify off-chain clients following changes to the contract.

```
2022-12-escher/src/Escher.sol::21 => function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/Escher721.sol::64 => function setDefaultRoyalty(
2022-12-escher/src/minters/FixedPriceFactory.sol::42 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/minters/LPDAFactory.sol::46 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/minters/OpenEditionFactory.sol::42 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/uris/Base.sol::10 => function setBaseURI(string memory _baseURI) external onlyOwner {
2022-12-escher/src/uris/Generative.sol::14 => function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {
2022-12-escher/src/uris/Generative.sol::19 => function setSeedBase() external onlyOwner {
```

## [N-04] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

instances:

```
2022-12-escher/src/Escher.sol::11 => bytes32 public constant CREATOR_ROLE = keccak256("CREATOR_ROLE");
2022-12-escher/src/Escher.sol::13 => bytes32 public constant CURATOR_ROLE = keccak256("CURATOR_ROLE");
2022-12-escher/src/Escher721.sol::17 => bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");
2022-12-escher/src/Escher721.sol::19 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
