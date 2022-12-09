- functions "setTokenRoyalty" is not implemented
[See №2](https://github.com/code-423n4/2022-12-escher#sales-patterns) in Fixed Price,  Open Edition and Last Price Dutch Auction (LPDA)
- No ```amount``` parameter is used as said in docs, used ```finalId```.
[See №3](https://github.com/code-423n4/2022-12-escher#fixed-price)
Change docs or variables names for clarity.
```
src\minters\FixedPrice.sol:
  14      struct Sale {
  15:         // slot 1 
  16:         uint48 currentId; 
  17:         uint48 finalId;
  18:         address edition; 
  19:         // slot 2 
  20:         uint96 price; 
  21:         address payable saleReceiver;  
  22:         // slot 3 
  23:         uint96 startTime; 
  24      }
```
- Change to ```saleTime``` in code or to ```startTime``` in docs
[See №3](https://github.com/code-423n4/2022-12-escher#fixed-price)
```
src\minters\FixedPrice.sol:
  23:         uint96 startTime; 

src\minters\OpenEdition.sol:
  19:         uint96 startTime; 
```
- Use consistent names in ```buy()``` function (sale_ or temp)
```
src\minters\FixedPrice.sol:
  58:         Sale memory sale_ = sale; 

src\minters\LPDA.sol:
  60:         Sale memory temp = sale;  

src\minters\OpenEdition.sol:
  60:         Sale memory temp = sale;
```

- Check that amount > 0 because it is possible to buy 0 nft even though none will be minted
```
src\minters\FixedPrice.sol:
  57:     function buy(uint256 _amount) external payable {

src\minters\LPDA.sol:
  58:      function buy(uint256 _amount) external payable {

src\minters\OpenEdition.sol:
  58:      function buy(uint256 _amount) external payable {
```
- It is possible to buy editions in LPDA after auction ends
- grantRole() is internal but according to docs should be public
[See №4](https://github.com/code-423n4/2022-12-escher#fixed-price)
```
src\Escher.sol:
  60:     function _grantRole(bytes32 _role, address _account) internal override { 
```
- Change parameter type to uint24 (uint48) and remove type casting to avoid disinformation
```
src\minters\FixedPrice.sol:
  57:     function buy(uint256 _amount) external payable {

  62:         uint48 newId = uint48(_amount) + sale_.currentId; 
  
src\minters\LPDA.sol:
  58:     function buy(uint256 _amount) external payable {
  59          uint48 amount = uint48(_amount); 

src\minters\OpenEdition.sol:
  58:     function buy(uint256 _amount) external payable { 
  59          uint24 amount = uint24(_amount); 
```
- Place ```initialize``` function right after constructor
```
src\minters\FixedPrice.sol:
  78:     function initialize(Sale memory _sale) public initializer {
```
- Contract names ```FixedPrice.sol``` and ```FixedPriceFactory.sol``` does not match documentation
Should be ```FixedPriceSale.sol``` and ```FixedPriceSaleFactory.sol``` [See](https://github.com/code-423n4/2022-12-escher#fixedpricesalesoll).
- Misleading comment
```
src\minters\FixedPrice.sol:
  12:     /// store nextId and remainingSupply, where nextId increases and remainingSupply decreases to 0 
```
- Use Checks Effects Interactions pattern
It is best practice to use [this pattern](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html). 
Place ```sale.currentId = newId;``` before loop with external code
```
src\minters\FixedPrice.sol:
  65:         for (uint48 x = sale_.currentId + 1; x <= newId; x++) { 
  66:             nft.mint(msg.sender, x); 
  67:         } 
  69:         sale.currentId = newId;  

src\minters\LPDA.sol: 
  73:         for (uint256 x = temp.currentId + 1; x <= newId; x++) { 
  74:             nft.mint(msg.sender, x); 
  75:         } 
  77:         sale.currentId = newId;  

src\minters\OpenEdition.sol: 
  67:         for (uint24 x = temp.currentId + 1; x <= newId; x++) { 
  68:             nft.mint(msg.sender, x); 
  69:         } 
  70:         sale.currentId = newId; 
```
- Missing NatSpec - 14 instances
```
src\Escher.sol:
  31:     function supportsInterface(
  60:     function _grantRole(bytes32 _role, address _account) internal override { 
  67:     function _revokeRole(bytes32 _role, address _account) internal override {

src\Escher721.sol: 
  84:     function supportsInterface(

src\minters\LPDA.sol: 
  142:     function lowestPrice() public view returns (uint256) {   
  146:     function _end() internal {

src\minters\OpenEdition.sol:
  117:     function available() public view returns (uint256) {
  121:     function _end(Sale memory _sale) internal {

src\uris\Base.sol:
  10:     function setBaseURI(string memory _baseURI) external onlyOwner { 
  14:     function tokenURI(uint256) external view virtual returns (string memory) {
  18:     function initialize(address _owner) public initializer {

src\uris\Generative.sol:  
  19:     function setSeedBase() external onlyOwner {  
  27:     function tokenToSeed(uint256 _tokenId) internal view returns (bytes32) { 

src\uris\Unique.sol: 
  10:     function tokenURI(uint256 _tokenId) external view virtual override returns (string memory) { 
```