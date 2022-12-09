# Floating pragma.

### Line Of Code:

All file

### Vulnerability and recommended fix

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;
```

We recommend the project use the fix version instead of fixing version.

# lack of address(0) validation.

### Line Of Code:

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L42

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L42

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L46

### Vulnerability and recommended fix

When setting the fee receiver, if the owner set the fee receiver to address(0), the 5% of the fee is lost.

```solidity
/// @notice set the fee receiver for fixed price editions
/// @param fees the address to receive fees
function setFeeReceiver(address payable fees) public onlyOwner {
	feeReceiver = fees;
}
```

# Lack of two step ownership transfer pattern.

### Line Of Code:

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L9

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L9

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L9

### Vulnerability and recommended fix

The factory contract inherits ownable from openzepplein.

```solidity
contract FixedPriceFactory is Ownable {
```

In the Ownable, the owenr is transferred by calling 

```solidity
function transferOwnership(address newOwner) public virtual onlyOwner {
	require(newOwner != address(0), "Ownable: new owner is the zero address");
	_transferOwnership(newOwner);
}
```

but we are not sure if the new owner is able to claiming the onlyOwner related function.

We recommend the project implement two step transfer to let the new owner claim the ownership.

# Missing Event emission.

### Line Of Code:

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L42

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L42

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L46

### Vulnerability and recommended fix.

```solidity
/// @notice set the fee receiver for fixed price editions
/// @param fees the address to receive fees
function setFeeReceiver(address payable fees) public onlyOwner {
	feeReceiver = fees;
}
```

changing fee receiver changes smart contract states and update the fee receiver address, which should emit event for indexer to capture the state change.

# Missing calling _init() function call when using upgradeable contract

### Line Of Code:

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L10

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L10

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L10

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L14

### Vulnerability and recommended fix.

The Escher721 inherits from ERC2981Upgradeable.

```solidity
contract Escher721 is
    Initializable,
    ERC721Upgradeable,
    AccessControlUpgradeable,
    ERC2981Upgradeable
{
```

then the Escher721 should call __ERC2981_init() in initialize function.

```solidity
    function initialize(
        address _creator,
        address _uri,
        string memory _name,
        string memory _symbol
    ) public initializer {
        __ERC721_init(_name, _symbol);
        __AccessControl_init();

        tokenUriDelegate = _uri;

        _grantRole(DEFAULT_ADMIN_ROLE, _creator);
        _grantRole(MINTER_ROLE, _creator);
        _grantRole(URI_SETTER_ROLE, _creator);
    }
```

OpenEdition.sol, LPDA.sol and FixedPrice.sol inherits from OwnableUpgradeable,

then the contract should call in the initializer function to make sure the owner is properly set.

```solidity
__Ownable_init()
```


# Lack of timestamp input validation when setting the sale parameter.

### Line Of Code:

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L79

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L111

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L83

### Vulnerability and recommended fix.

When creating sales using FixedPrice.sol and OpenEdition.sol and LPDA.sol 

Using needs to set the start time and end time

```solidity
struct Sale {
	// slot 1
	uint72 price;
	uint24 currentId;
	address edition;
	// slot 2
	uint96 startTime;
	address payable saleReceiver;
	// slot 3
	uint96 endTime;
}
```

The code should check if the block.timestamp < startTime < endTime.

start time and end time should not be in the past and start time should be earlier than endTime, otherwise, the buy function would not work properly.




