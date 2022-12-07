


### [L01] Missing checks for `address(0x0)` when assigning values to `address` state variables


#### Findings:
```
2022-12-escher/src/minters/FixedPrice.sol::79 => sale = _sale;
2022-12-escher/src/minters/LPDA.sol::111 => sale = _sale;
2022-12-escher/src/minters/OpenEdition.sol::83 => sale = _sale;
2022-12-escher/src/uris/Base.sol::11 => baseURI = _baseURI;
2022-12-escher/src/uris/Generative.sol::16 => scriptPieces[_id] = _data;
```



### [L02] `initialize` functions can be front-run

#### Impact
See [this](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/40) finding from a prior badger-dao contest for details
#### Findings:
```
2022-12-escher/src/Escher721.sol::32 => function initialize(
2022-12-escher/src/Escher721.sol::37 => ) public initializer {
2022-12-escher/src/minters/FixedPrice.sol::78 => function initialize(Sale memory _sale) public initializer {
2022-12-escher/src/minters/LPDA.sol::110 => function initialize(Sale calldata _sale) public initializer {
2022-12-escher/src/minters/OpenEdition.sol::82 => function initialize(Sale memory _sale) public initializer {
2022-12-escher/src/uris/Base.sol::18 => function initialize(address _owner) public initializer {
```



### [L03] Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions

#### Impact
See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.
#### Findings:
```
2022-12-escher/src/minters/FixedPrice.sol::10 => contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
2022-12-escher/src/minters/LPDA.sol::10 => contract LPDA is Initializable, OwnableUpgradeable, ISale {
2022-12-escher/src/minters/OpenEdition.sol::10 => contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
2022-12-escher/src/uris/Base.sol::7 => contract Base is ITokenUriDelegate, OwnableUpgradeable {
```



### [L04] `address.call{value:x}()` should be used instead of `payable.transfer()`

#### Impact
The use of `payable.transfer()` is heavily frowned upon because it can lead to the locking of funds. The `transfer()` call requires that the recipient has a `payable` callback, only provides 2300 gas for its operation. This means the following cases can cause the transfer to fail:

- The contract does not have a `payable` callback
- The contract’s `payable` callback spends more than 2300 gas (which is only enough to emit something)
- The contract is called through a proxy which itself uses up the 2300 gas
#### Findings:
```
2022-12-escher/src/minters/LPDA.sol::105 => payable(msg.sender).transfer(owed);
```






### [L05] Unspecific Compiler Version Pragma

#### Impact
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.
#### Findings:
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








#### Non-Critical Issues





### [N01] Adding a return statement when the function defines a named return variable, is redundant


#### Findings:
```
2022-12-escher/src/uris/Base.sol::15 => return baseURI;
```





### [N02] Event is missing `indexed` fields

#### Impact
Each event should use three indexed fields if there are three or more fields
#### Findings:
```
2022-12-escher/src/minters/FixedPrice.sol::40 => event Buy(address indexed _buyer, uint256 _amount, uint256 _value, Sale _saleInfo);
2022-12-escher/src/minters/LPDA.sol::43 => event Buy(address indexed _buyer, uint256 _amount, uint256 _value, Sale _saleInfo);
2022-12-escher/src/minters/OpenEdition.sol::39 => event Buy(address indexed _buyer, uint256 _amount, uint256 _value, Sale _saleInfo);
```




### [N03] Unused file


#### Findings:
```
2022-12-escher/src/Escher.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/Escher721.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/Escher721Factory.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/minters/FixedPrice.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/minters/FixedPriceFactory.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/minters/LPDA.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/minters/LPDAFactory.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/minters/OpenEdition.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/minters/OpenEditionFactory.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/uris/Base.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/uris/Generative.sol::1 => // SPDX-License-Identifier: MIT
2022-12-escher/src/uris/Unique.sol::1 => // SPDX-License-Identifier: MIT
```


### [N04] `public` functions not called by the contract should be declared `external` instead

#### Impact
Contracts are allowed to override their parents’ functions and change the visibility from public to external .
#### Findings:
```
2022-12-escher/src/Escher.sol::21 => function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/Escher.sol::27 => function addCreator(address _account) public onlyRole(CURATOR_ROLE) {
2022-12-escher/src/Escher721.sol::37 => ) public initializer {
2022-12-escher/src/Escher721.sol::51 => function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {
2022-12-escher/src/Escher721.sol::57 => function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {
2022-12-escher/src/Escher721.sol::67 => ) public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/Escher721.sol::72 => function resetDefaultRoyalty() public onlyRole(DEFAULT_ADMIN_ROLE) {
2022-12-escher/src/minters/FixedPrice.sol::78 => function initialize(Sale memory _sale) public initializer {
2022-12-escher/src/minters/FixedPrice.sol::86 => function getPrice() public view returns (uint256) {
2022-12-escher/src/minters/FixedPrice.sol::91 => function startTime() public view returns (uint256) {
2022-12-escher/src/minters/FixedPrice.sol::96 => function editionContract() public view returns (address) {
2022-12-escher/src/minters/FixedPrice.sol::101 => function available() public view returns (uint256) {
2022-12-escher/src/minters/FixedPriceFactory.sol::42 => function setFeeReceiver(address payable fees) public onlyOwner {
2022-12-escher/src/minters/LPDA.sol::99 => function refund() public {
2022-12-escher/src/minters/LPDA.sol::110 => function initialize(Sale calldata _sale) public initializer {
2022-12-escher/src/minters/LPDA.sol::117 => function getPrice() public view returns (uint256) {
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



### [N05] Constant redefined elsewhere

#### Impact
Consider defining in only one contract so that values cannot become out of sync when only one location is updated
#### Findings:
```
2022-12-escher/src/Escher.sol::11 => bytes32 public constant CREATOR_ROLE = keccak256("CREATOR_ROLE");
2022-12-escher/src/Escher.sol::13 => bytes32 public constant CURATOR_ROLE = keccak256("CURATOR_ROLE");
2022-12-escher/src/Escher721.sol::17 => bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");
2022-12-escher/src/Escher721.sol::19 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```

