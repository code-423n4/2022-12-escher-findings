# [L] ADD TWO STEP TRANSFER OWNERSHIP PATTERN

The project using Openzeppelin `Ownable` contract for ownership. The implementation of openzeppelin ownable contract doesn't handle two step verification owner transfer.

All contracts are inherited from OpenZeppelin’s Ownable and OwnableUpgradable contract which enables the onlyOwner role to transfer ownership to another address. It’s possible that the onlyOwner role mistakenly transfers ownership to the wrong address, resulting in a loss of the onlyOwner role.

If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the onlyOwner() modifier. Lack of two-step procedure for critical operations leaves them error-prone if the address is incorrect, the new address will take on the functionality of the new role immediately

# [NC] NO SAME VALUE INPUT CONTROL

It's better to have a same value input control so if no changes it will just revert, saving gas

```solidity
File: Escher.sol
21:     function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
22:         _setURI(_newuri);
23:     }
...
27:     function addCreator(address _account) public onlyRole(CURATOR_ROLE) {
28:         _grantRole(CREATOR_ROLE, _account);
29:     }
```

```solidity
File: Base.sol
10:     function setBaseURI(string memory _baseURI) external onlyOwner {
11:         baseURI = _baseURI;
12:     }
```

```solidity
File: FixedPriceFactory.sol
42:     function setFeeReceiver(address payable fees) public onlyOwner {
43:         feeReceiver = fees;
44:     }
```

# [L] NO CHECK FOR INPUT VALIDATION

No check for address(0) when changing state variable `feeReceiver`

```solidity
File: FixedPriceFactory.sol
42:     function setFeeReceiver(address payable fees) public onlyOwner {
43:         feeReceiver = fees;
44:     }
```

# [NC] EVENT SHOULD BE EMITTED IN SETTERS


Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

```solidity
File: FixedPriceFactory.sol
42:     function setFeeReceiver(address payable fees) public onlyOwner {
43:         feeReceiver = fees;
44:     }
```