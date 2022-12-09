## Two-step Transfer of Ownership
In `Base.sol`, the contract inherits from `OwnableUpgradeable` where `_transferOwnership()` is invoked in `initialize()` allowing the oldOwner to transfer ownership to the newOwner. This function does not implement zero address check (like `transferOwnership` does) and proceeds directly to write the new owner's address into the owner's state variable, `_owner`. If the nominated account is not a valid account, it is entirely possible the contract deployer may accidentally transfer ownership to an uncontrolled account or a zero address, breaking all functions with the onlyOwner() modifier.

Consider implementing a two step process where the deployer nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed (using OpenZeppelin's `Ownable2StepUpgradeable.sol` if need be).  This will ensure the new owner is going to be fully aware of the ownership assigned/transferred other than having the above mistake avoided.

All other instances entailed:

[File: FixedPrice.sol#L80](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L80)

```
        _transferOwnership(_sale.saleReceiver);
```
[File: OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

```
        _transferOwnership(sale.saleReceiver);
```
[File: LPDA.sol#L112](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L112)

```
        _transferOwnership(sale.saleReceiver);
```
## Shadowed Variable
The `_owner` state variable inherited from `OwnableUpgradeable.sol` is shadowed in `initialize()` of `Base.sol`.

[File: Base.sol#L18-L20](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L18-L20)

```
    function initialize(address _owner) public initializer {
        _transferOwnership(_owner);
    }
```
Consider renaming this parameter to `owner_` to avoid shadowing the inherited variable.

## Add a Constructor Initializer
As per Openzeppelin's recommendation:

https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/6

The guidelines are now to prevent front-running of initialize() on an implementation contract, by adding an empty constructor with the initializer modifier. Hence, the implementation contract gets initialized atomically upon deployment.

This feature is readily incorporated in the Solidity Wizard since the UUPS vulnerability discovery. You would just need to check UPGRADEABILITY to have the following constructor code block added to the contract:

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }
Here are the contract instances entailed:

[File: Base.sol#L18](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L18)

```
    function initialize(address _owner) public initializer {
```
[File: Escher721.sol#L25](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L25)

```
    constructor() {}
```
## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

Here are the contract instances lacking NatSpec mostly in its entirety:

[File: Unique.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol)
[File: Base.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol)

Here are the contract instances partially lacking NatSpec:

[File: Generative.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol)
[File: Escher721Factory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol)
[File: OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)
[File:LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol)
[File: Escher.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol)
[File: Escher721.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol)
[File: OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)
[File: LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

## Complementary Codehash Checks
Consider implementing an optional codehash check for immutable contract addresses at the constructor that will assure matching inputs of constructor parameters.

For instance, the following code block may be refactored as follows:

[File: Escher721Factory.sol#L17-L18](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol#L17-L18)

```
-    constructor(address _escher) {
+    constructor(address _escher, bytes32 _escherCodeHash) {  
+        require(_escher.codehash == _escherCodeHas, "INVALID ADDRESS");
        escher = Escher(_escher);
        ...
```
## Lack of Events for Critical Operations
Critical operations not triggering events will make it difficult to review the correct behavior of the deployed contracts. Users and blockchain monitoring systems will not be able to detect suspicious behaviors at ease without events. Consider adding events where appropriate for all critical operations for better support of off-chain logging API.

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
[File: FixedPriceFactory.sol#L31](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L31)

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
[File: Escher721.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol)

```
57:    function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {
```
## Constructor of Escher.sol
According to [README.md](https://github.com/code-423n4/2022-12-escher), only assigned roles as a Curator or a Creator are ERC1155 soulbound tokens. However, the contract deployer, `msg.sender`, would also join the party as a soulbound token that could create confusion to the users and the protocol.

Consider having the function call in the constructor refactored as follows:

[File: Escher.sol#L15-L67](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L15-L16)

```
    constructor() ERC1155("") {
-        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
+        super._grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
```
## CURATOR_ROLE in Escher.sol
No default `CURATOR_ROLE` was assigned upon the contract deployment of `Escher.sol` despite this could separately be done outside the contract. 

Consider having the following code line added to the [constructor](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L15-L17), serving also as an emergency measure by allowing the contract owner to assume the curator role when need be:

```
        _grantRole(CURATOR_ROLE, msg.sender);
```
Additionally, it is recommended implementing `addCurator()` in the contract that could look something as follows:

```
    function addCurator(address _account) public onlyRole(DEFAULT_ADMIN_ROLE) {
        _grantRole(CURATOR_ROLE, _account);
    }
```
## Inherited Calls to AccessControl.sol
Throughout the code bases, it is noted that the protocol always skip the preliminary functions when making inherited calls to `AccessControl.sol`. For instance, it would call `_transferOwnership()` instead of `transferOwnership()`, `_grantRole()` instead of `grantRole()`, and `_revokeRole()` instead of `revokeRole()`.

While it is understandable that the adoption of `_grantRole()` would free up more roles other than the admin to carry out their respective duties, care must be taken for the other two calls.

As earlier explained at the top of this report, `_transferOwnership()` skips zero address check which is duly acknowledged as outside of the scope of this contest.

[`_revokeRole()`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L68), as the name of the function suggests, could sometimes trick developer into thinking an admin modifier would have been taken care of at `AccessControl.sol` when it is not. It is recommended using `revokeRole()` as a double measure to better prevent this critical call from being invoked by anyone.  

## `_safeMint()` Over `_mint()` for ERC721 Tokens
Consider using `_safeMint()` instead of `_mint()` since the former would determine whether or not the recipient is a contract that has a logic implemented to support ERC721 standard via `onERC721Received()`. This is to prevent any token from getting trapped in a smart contract that cannot feature future token transfer.

Here is the instance entailed:

[File: Escher721.sol#L52](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L52)

```
        _mint(to, tokenId);
```
