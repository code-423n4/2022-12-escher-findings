## Table of Contents
### Low Risk Issues
- Use _safeMint instead of _mint
- Missing Zero Address Check 
- Use fixed compiler versions instead of floating version

### Non-critical Issues
- Wrong NatSpec comment
- Unnecessary inheritance
- Define Magic Numbers to Constant

&ensp;
## Low Risk Issues
### Use _safeMint instead of _mint

#### Issue
Use of _mint is discouraged and recommended to use _safeMint whenever possible by OpenZeppelin.
This is because _mint cannot check whether the receiving address know how to handle ERC721 tokens.    

In the function shown at below PoC, ERC721 token is sent to msg.sender with the _mint method.
If this msg.sender is a contract and is not aware of incoming ERC721 tokens, the sent token can be 
locked up in the contract forever.

#### PoC
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L51-L53
```solidity
    function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {
        _mint(to, tokenId);
    }
```
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65-L67
```solidity
        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }
```
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66-L68
```solidity
        for (uint24 x = temp.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }
```
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73-L75
```solidity
        for (uint256 x = temp.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }
```

#### Mitigation
I recommend to call the _safeMint() method instead of _mint().

&ensp;
### Missing Zero Address Check 

#### Issue
I recommend adding check of 0-address for input validation of critical address parameters.
Not doing so might lead to non-functional contract and have to redeploy the contract, when it is updated to 0-address accidentally.

#### PoC
Total of 6 instances found.
1. Escher721Factory.sol:constructor():  `escher` address 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L17-L18

2. Escher721Factory.sol:createContract():  `_uri` address 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L35

3. Escher.sol:addCreator():  `_account` address 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L28

4. LPDAFactory.sol:setFeeReceiver():  `fees` address 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L46-L47

5. OpenEditionFactory.sol:setFeeReceiver():  `fees` address 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L42-L43

6. FixedPriceFactory.sol:setFeeReceiver():  `fees` address 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L42-L43

#### Mitigation
Add 0-address check for above addresses.

&ensp;
### Use fixed compiler versions instead of floating version

#### Issue
It is best practice to lock your pragma instead of using floating pragma.
The use of floating pragma has a risk of accidentally get deployed using latest complier
which may have higher risk of undiscovered bugs.
Reference: https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/

#### PoC
Total of 12 instances found.
```
./LPDAFactory.sol:2:pragma solidity ^0.8.17;
./FixedPriceFactory.sol:2:pragma solidity ^0.8.17;
./Escher.sol:2:pragma solidity ^0.8.17;
./OpenEdition.sol:2:pragma solidity ^0.8.17;
./Unique.sol:2:pragma solidity ^0.8.17;
./OpenEditionFactory.sol:2:pragma solidity ^0.8.17;
./Generative.sol:2:pragma solidity ^0.8.17;
./Base.sol:2:pragma solidity ^0.8.17;
./LPDA.sol:2:pragma solidity ^0.8.17;
./FixedPrice.sol:2:pragma solidity ^0.8.17;
./Escher721Factory.sol:2:pragma solidity ^0.8.17;
./Escher721.sol:2:pragma solidity ^0.8.17;
```

#### Mitigation
I suggest to lock your pragma and aviod using floating pragma.
```
// bad
pragma solidity ^0.8.10;

// good
pragma solidity 0.8.10;
```

&ensp;
## Non-critical Issues
### Wrong NatSpec Comments

#### Issue
Following NatSpec has incorrect comments. 

#### PoC
NatSpec on Line 84 of FixedPrice.sol is not relevant. Delete this line.
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L84
```solidity
84:    /// @notice cancel a fixed price sale
85:    /// @notice the price of the sale
86:    function getPrice() public view returns (uint256) {
87:        return sale.price;
88:    }
```

&ensp;
### Unnecessary inheritance

#### Issue
There is an unnecessary contract inheritance.

#### PoC
1. FixedPrice.sol
Initializable is unnecessary inheritance since OwnableUpgradeable already inherit Initializable.
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L10
```solidity
File: FixedPrice.sol
10:contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
```
```solidity
File: OwnableUpgradeable.sol
21:abstract contract OwnableUpgradeable is Initializable, ContextUpgradeable {
```

&ensp;
### Define Magic Numbers to Constant

#### Issue
It is best practice to define magic numbers to constant rather than just using as a magic number.
This improves code readability and maintainability. 

#### PoC
1. Magic Number: 20
```solidity
./OpenEdition.sol:92:        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
./LPDA.sol:84:            uint256 fee = totalSale / 20;
./FixedPrice.sol:109:        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```

#### Mitigation
Define magic numbers to constant.

&ensp;
