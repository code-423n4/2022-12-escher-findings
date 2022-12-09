### [G-01] Empty blocks should be removed or emit something


#### Impact
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.


#### Findings:
```
src/Escher721.sol:L25    constructor() {}

```
        
### [G-02] Functions guaranteed to revert when called by normal users can be declared as payable.


#### Impact
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.


#### Findings:
```
src/Escher.sol:L21    function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {

src/Escher.sol:L27    function addCreator(address _account) public onlyRole(CURATOR_ROLE) {

src/Escher721.sol:L51    function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {

src/Escher721.sol:L57    function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {

src/Escher721.sol:L72    function resetDefaultRoyalty() public onlyRole(DEFAULT_ADMIN_ROLE) {

src/minters/OpenEdition.sol:L75    function cancel() external onlyOwner {

src/minters/FixedPrice.sol:L50    function cancel() external onlyOwner {

src/minters/LPDA.sol:L92    function cancel() external onlyOwner {

src/uris/Base.sol:L10    function setBaseURI(string memory _baseURI) external onlyOwner {

src/uris/Generative.sol:L14    function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {

src/uris/Generative.sol:L19    function setSeedBase() external onlyOwner {

```

### [G-03] ```X += Y``` costs more gas than ```X = X + Y``` for state variables.


#### Impact
Consider changing ```X += Y``` to ```X = X + Y``` to save gas.


#### Findings:
```
src/minters/LPDA.sol:L66        amountSold += amount;

src/minters/LPDA.sol:L70        receipts[msg.sender].amount += amount;

src/minters/LPDA.sol:L71        receipts[msg.sender].balance += uint80(msg.value);

```

