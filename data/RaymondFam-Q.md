## Two-step Transfer of Ownership
In `Base.sol`, the contract inherits from `OwnableUpgradeable` where `_transferOwnership()` is invoked in `initialize()` allowing the oldOwner to transfer ownership to the newOwner. This function does not implement zero address check (like `transferOwnership` does) and proceeds directly to write the new owner's address into the owner's state variable, `_owner`. If the nominated EOA account is not a valid account, it is entirely possible the contract deployer may accidentally transfer ownership to an uncontrolled account or a zero address, breaking all functions with the onlyOwner() modifier.

Consider implementing a two step process where the deployer nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed.  This will ensure the new owner is going to be fully aware of the ownership assigned/transferred other than having the above mistake avoided.

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
Here are some of the contract instances entailed:

[File: Base.sol#L18](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L18)

```
    function initialize(address _owner) public initializer {
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
