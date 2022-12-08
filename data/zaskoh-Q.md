# solidity pragma versioning
At the moment ^0.8.17 is used for all contracts. This can be changed to 0.8.17 as they're not libraries and it's better to use a fixed version for the deployment.

# remove unneccessari function / set variable private

```solidity
File: src/uris/Base.sol
8:     string public baseURI; // @audit set private or remove function tokenURI - redundant
```

# constants should be defined rather than using magic numbers
It is bad practice to use numbers directly in code without explanation.

```solidity
File: src/minters/FixedPrice.sol
109:         ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20); // @audit constants should be defined rather than using magic numbers

File: src/minters/LPDA.sol
84:             uint256 fee = totalSale / 20; // @audit constants should be defined rather than using magic numbers

File: src/minters/OpenEdition.sol
92:         ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20); // @audit constants should be defined rather than using magic numbers

File: src/uris/Generative.sol
24:         seedBase = keccak256(abi.encodePacked(numb, blockhash(numb - 1), time, (time % 200) + 1)); // @audit constants should be defined rather than using magic numbers
```

# missing checks for address that should not be address(0)
Some addresses should be checked before deployment as they're not changeable after the deployment and user (creator) funds can be lost
```solidity
File: src/minters/FixedPriceFactory.sol
29:     function createFixedSale(FixedPrice.Sale calldata _sale) external returns (address clone) {
30:         require(IEscher721(_sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
31:         require(_sale.startTime >= block.timestamp, "START TIME IN PAST");
32:         require(_sale.finalId > _sale.currentId, "FINAL ID BEFORE CURRENT");
33:         // @audit check for _sale.saleReceiver != address(0) as this is used for the payment after the selling is finished
34:         // @audit also check for _sale.edition != address(0) as this address is used for minting without a success check

File: src/minters/LPDAFactory.sol
29:     function createLPDASale(LPDA.Sale calldata sale) external returns (address clone) {
30:         require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
31:         require(sale.saleReceiver != address(0), "INVALID SALE RECEIVER");
32:         require(sale.startTime >= block.timestamp, "INVALID START TIME");
33:         require(sale.endTime > sale.startTime, "INVALID END TIME");
34:         require(sale.finalId > sale.currentId, "INVALID FINAL ID");
35:         require(sale.startPrice > 0, "INVALID START PRICE");
36:         require(sale.dropPerSecond > 0, "INVALID DROP PER SECOND");
37:         // @audit also check for sale.edition as this address is used for minting and should not be zero

File: src/minters/OpenEditionFactory.sol
29:     function createOpenEdition(OpenEdition.Sale calldata sale) external returns (address clone) {
30:         require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
31:         require(sale.startTime >= block.timestamp, "START TIME IN PAST");
32:         require(sale.endTime > sale.startTime, "END TIME BEFORE START");
33:         // @audit check for _sale.saleReceiver != address(0) as this is used for the payment after the selling is finished
34:         // @audit also check for _sale.edition != address(0) as this address is used for minting without a success check

```

# no transfer ownership pattern (two-phase ownsership transfer)
Recommend using Ownable2Step from OpenZeppelin for a two step process where the owner or admin nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

# add missing documentation
The contracts are very well documented and clear to read. Still it misses some documentation that could be added for a even better developer experience.
Functions missing @param (or wrong param mentioned) / @return / @notice or is missing complete

```solidity
File: src/minters/FixedPrice.sol
83: 
84:     /// @notice cancel a fixed price sale // @audit wrong comment, can be removed
85:     /// @notice the price of the sale
86:     function getPrice() public view returns (uint256) {

File: src/minters/FixedPriceFactory.sol
14:     // @audit add NatSpec
15:     event NewFixedPriceContract(

File: src/minters/FixedPriceFactory.sol
26:     // @audit missing @return in NatSpec
27:     /// @notice create a fixed sale proxy contract
28:     /// @param _sale the sale info
29:     function createFixedSale(FixedPrice.Sale calldata _sale) external returns (address clone) {

File: src/minters/LPDA.sol
41:     event Start(Sale _saleInfo); // @audit add NatSpec
42:     event End(Sale _saleInfo); // @audit add NatSpec
43:     event Buy(address indexed _buyer, uint256 _amount, uint256 _value, Sale _saleInfo); // @audit add NatSpec

File: src/minters/LPDAFactory.sol
15:     event NewLPDAContract( // @audit add NatSpec

File: src/minters/LPDAFactory.sol
26:     // @audit missing @return in NatSpec
27:     /// @notice create a fixed sale proxy contract
28:     /// @param sale the sale info
29:     function createLPDASale(LPDA.Sale calldata sale) external returns (address clone) {

File: src/minters/OpenEdition.sol
116:     function available() public view returns (uint256) {  // @audit add NatSpec

File: src/minters/OpenEditionFactory.sol
15:     event NewOpenEditionContract( // @audit add NatSpec

File: src/minters/OpenEditionFactory.sol
26:     // @audit missing @return NatSpec
27:     /// @notice create a fixed sale proxy contract
28:     /// @param sale the sale info
29:     function createOpenEdition(OpenEdition.Sale calldata sale) external returns (address clone) {

File: src/uris/Base.sol
10:     function setBaseURI(string memory _baseURI) external onlyOwner { // @audit add NatSpec

File: src/uris/Base.sol
14:     function tokenURI(uint256) external view virtual returns (string memory) { // @audit add NatSpec

File: src/uris/Base.sol
18:     function initialize(address _owner) public initializer { // @audit add NatSpec

File: src/uris/Generative.sol
19:     function setSeedBase() external onlyOwner { // @audit add NatSpec

File: src/uris/Unique.sol
10:     function tokenURI(uint256 _tokenId) external view virtual override returns (string memory) { // @audit add NatSpec

File: src/Escher.sol
31:     function supportsInterface( // @audit add NatSpec

File: src/Escher721.sol
84:     function supportsInterface( // @audit add NatSpec

File: src/Escher721Factory.sol
15:     event NewEscher721Contract(address indexed creator, address indexed clone, address indexed uri); // @audit add NatSpec

File: src/Escher721Factory.sol
22:     // @audit add @return NatSpec
23:     /// @notice create a new escher unique contract
24:     /// @param _name the name of the contract
25:     /// @param _symbol the symbol of the contract
26:     /// @param _uri the uri delegate contract
27:     function createContract(
```

# typos
There is a typo in the comments.

```solidity
File: src/Escher.sol
19:     /// @notice Updates the metadata for Escher Solbound token // @audit typo Solbound => Soulbound
20:     /// @param _newuri The new metadata URI
21:     function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
```

