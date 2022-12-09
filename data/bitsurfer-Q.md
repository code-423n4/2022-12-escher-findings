# NO CHECK FOR INPUT VALIDATION

There is no check for address(0) when changing state variable `feeReceiver`

FixedPriceFactory.sol
```solidity
42:     function setFeeReceiver(address payable fees) public onlyOwner {
43:         feeReceiver = fees;
44:     }
```

# NO TRANSFER OWNERSHIP PATTERN

Openzeppelin Ownership contract is not using two step transfer pattern. This function checks that the new owner is not the zero address and proceeds to write the new owner’s address into the owner’s state variable. If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the onlyOwner() modifier.

Recommend considering implementing a two step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

# NO EMIT WHEN CHANGING STATE

When changing state, it's best practice to emit an event.
Most of contract in this project did emit an event, but this following functions are missing 

FixedPriceFactory.sol
```solidity
42:     function setFeeReceiver(address payable fees) public onlyOwner {
43:         feeReceiver = fees;
44:     }
```