|  | Issue | Instances |
| --- | --- | --- |
| [L-01] | Different emit event behavior in buy() functions | 2 |
| [N-01] | Non-library/interface files should use fixed compiler versions, not floating ones. | 4 |
| [N-02] | Function order is incorrect. | 3 |
| [N-03] | Internal function should use prefix _ | 1 |
| [N-04] | _ in front of function parameters. | 9 |
| [N-05] | Missing NatSpec comments | 44 |
| [N-06] | Missing NatSpec comments for Function parameters | 3 |
| [N-07] | Wrong NatSpec @notice for buy() function | 1 |

## ****Low-Risk****

### [L-01] Different emit event behavior in `buy()` functions

In LPDA.sol and OpenEdition.sol `buy`() function emit Buy event with the cached version of Sale object with the old currentId (before minting). While in FixedPrice.sol `buy()` emit Buy event with Sale object from storage with changed currentId to newId.

```diff
file: src/minters/FixedPrice.sol

function buy(uint256 _amount) external payable {
		...
    emit Buy(msg.sender, _amount, msg.value, sale); <<- read sale from storage
		...
}
```

```diff
file: src/minters/LPDA.sol
&
file: src/minters/OpenEdition.sol

function buy(uint256 _amount) external payable {
		...
    Sale memory temp = sale; <<- Cache sale to temp
		...
    sale.currentId = newId; <<- Store newId to storage
		...
    emit Buy(msg.sender, amount, msg.value, temp); <<- Emit event with cached sale
		...
}
```

## ****Non-Critical Issues****

### [N-01] Non-library/interface files should use fixed compiler versions, not floating ones.

```diff
file: src/interfaces/IEscher721.sol

- pragma solidity ^0.8.17;
+ pragma solidity 0.8.17;
```

```diff
file: src/interfaces/ISale.sol

- pragma solidity ^0.8.17;
+ pragma solidity 0.8.17;
```

```diff
file: src/interfaces/ISaleFactory.sol

- pragma solidity ^0.8.17;
+ pragma solidity 0.8.17;
```

```diff
file: src/interfaces/ITokenUriDelegate.sol

- pragma solidity ^0.8.17;
+ pragma solidity 0.8.17;
```

### [N-02] Function order is incorrect, struct definition can not go after state variable declaration

```diff
file: src/minters/FixedPrice.sol

- address public immutable factory;
	/// store nextId and remainingSupply, where nextId increases and remainingSupply decreases to 0
	/// avoids strict equality of current == final
	struct Sale {
	    ...
	}
+ address public immutable factory;
```

```diff
file: src/minters/LPDA.sol

- address public immutable factory;
	/// we use different uints and some weird ordering to pack variables into 32 bytes
	struct Sale {
		...
	}
	struct Receipt {
		...
  }
+ address public immutable factory;
```

```diff
file: src/minters/OpenEdition.sol

- address public immutable factory;
	/// we use different uints and some weird ordering to pack variables into 32 bytes
	struct Sale {
	    ...
	}
+ address public immutable factory;
```

### [N-03] Internal function should use prefix `_`

Using an underscore in front of function parameters in Solidity is a convention that can help to make your code more readable and maintainable.

```diff
file: src/uris/Generative.sol

- function tokenToSeed(uint256 _tokenId) internal view returns (bytes32) {
+ function _tokenToSeed(uint256 _tokenId) internal view returns (bytes32) {
```

### [N-04] `_` in front of function parameters.

Using an underscore in front of function parameters in Solidity is a convention that can help to make your code more readable and maintainable.

```diff
file: src/Escher721.sol

- function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {
+ function mint(address _to, uint256 _tokenId) public virtual onlyRole(MINTER_ROLE) {

- function setDefaultRoyalty(address receiver, uint96 feeNumerator) public onlyRole(DEFAULT_ADMIN_ROLE) {
+ function setDefaultRoyalty(address _receiver, uint96 _feeNumerator) public onlyRole(DEFAULT_ADMIN_ROLE) {

- function supportsInterface(bytes4 interfaceId) ...
+ function supportsInterface(bytes4 _interfaceId) ...

- function _burn(uint256 tokenId) internal override(ERC721Upgradeable) {
+ function _burn(uint256 _tokenId) internal override(ERC721Upgradeable) {
```

```diff
file: src/minters/FixedPriceFactory.sol

- function setFeeReceiver(address payable fees) public onlyOwner {
+ function setFeeReceiver(address payable _fees) public onlyOwner {
```

```diff
file: src/minters/LPDAFactory.sol

- function createLPDASale(LPDA.Sale calldata sale) external returns (address clone) {
+ function createLPDASale(LPDA.Sale calldata _sale) external returns (address clone) {

- function setFeeReceiver(address payable fees) public onlyOwner {
+ function setFeeReceiver(address payable _fees) public onlyOwner {
```

```diff
file: src/minters/OpenEditionFactory.sol

- function createOpenEdition(OpenEdition.Sale calldata sale) external returns (address clone) {
+ function createOpenEdition(OpenEdition.Sale calldata _sale) external returns (address clone) {

- function setFeeReceiver(address payable fees) public onlyOwner {
+ function setFeeReceiver(address payable _fees) public onlyOwner {
```

### [N-05] Fully missing NatSpec comments

NatSpec comments are used to provide detailed documentation for a Solidity smart contract. This includes information about the contract's functions, variables, and overall purpose. This allows other developers to understand the contract's intended functionality and use it correctly.

```solidity
file: src/Escher.sol

function supportsInterface(bytes4 _interfaceId)
```

```solidity
file: src/Escher721.sol

contract Escher721 is Initializable, ERC721Upgradeable, AccessControlUpgradeable, ERC2981Upgradeable {
function supportsInterface(bytes4 interfaceId)
function _burn(uint256 tokenId) internal override(ERC721Upgradeable) {
```

```solidity
file: src/Escher721Factory.sol

contract Escher721Factory {
Escher public immutable escher;
address public immutable implementation;
event NewEscher721Contract(address indexed creator, address indexed clone, address indexed uri);
```

```solidity
file: src/uris/Base.sol

contract Base is ITokenUriDelegate, OwnableUpgradeable {
function setBaseURI(string memory _baseURI) external onlyOwner {
function tokenURI(uint256) external view virtual returns (string memory) {
function initialize(address _owner) public initializer {
```

```solidity
file: src/uris/Generative.sol

contract Generative is Unique {
bytes32 public seedBase;
function setSeedBase() external onlyOwner {
function tokenToSeed(uint256 _tokenId) internal view returns (bytes32) {
```

```solidity
file: src/uris/Unique.sol

contract Unique is Base {
function tokenURI(uint256 _tokenId) external view virtual override returns (string memory) {
```

```solidity
file: src/minters/FixedPrice.sol

contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
address public immutable factory;
```

```solidity
file: src/minters/FixedPriceFactory.sol

contract FixedPriceFactory is Ownable {
address payable public feeReceiver;
address public immutable implementation;
event NewFixedPriceContract(address indexed creator, address indexed edition, address indexed saleContract, FixedPrice.Sale saleInfo);
```

```solidity
file: src/minters/LPDA.sol

contract LPDA is Initializable, OwnableUpgradeable, ISale {
address public immutable factory;
uint48 public amountSold = 0;
event Start(Sale _saleInfo);
event End(Sale _saleInfo);
event Buy(address indexed _buyer, uint256 _amount, uint256 _value, Sale _saleInfo);
function lowestPrice() public view returns (uint256) {
function _end() internal {
```

```solidity
file: src/minters/LPDAFactory.sol

contract LPDAFactory is Ownable {
address payable public feeReceiver;
address public immutable implementation;
event NewLPDAContract(address indexed _creator, address indexed _edition, address indexed _saleContract, LPDA.Sale _saleInfo);
```

```solidity
file: src/minters/OpenEdition.sol

contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
address public immutable factory;
function available() public view returns (uint256) {
function _end(Sale memory _sale) internal {
```

```solidity
file: src/minters/OpenEditionFactory.sol

contract OpenEditionFactory is Ownable {
address payable public feeReceiver;
address public immutable implementation;
event NewOpenEditionContract(address indexed creator, address indexed edition, address indexed saleContract, OpenEdition.Sale saleInfo);
```

### [N-06] Missing NatSpec comments for Function parameters

NatSpec comments are used to provide detailed documentation for a Solidity smart contract. This includes information about the contract's functions, variables, and overall purpose. This allows other developers to understand the contract's intended functionality and use it correctly.

```solidity
file: src/Escher.sol

function _grantRole(bytes32 _role, address _account) internal override {
function _revokeRole(bytes32 _role, address _account) internal override {
```

```solidity
file: src/Escher721.sol

function _burn(uint256 tokenId) internal override(ERC721Upgradeable) {
```

### [N-07] Wrong NatSpec `@notice` for `buy()` function

Wrong `@notice` for `buy()` function since in LPDA.sol `msg.value` not fixed.

```diff
file: src/minters/LPDA.sol

/// @notice buy from a fixed price sale after the sale starts
/// @param _amount the amount of editions to buy
function buy(uint256 _amount) external payable {
```