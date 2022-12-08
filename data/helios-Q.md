# 1st report for : Unique.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol

## 1. Integer Overflow in tokenURI()

Summary:
The `tokenURI()` function in the `Unique` contract is vulnerable to an integer overflow attack because it uses the `toString()` function to convert a `uint256` value to a string. This function can produce unexpected results if the value exceeds the maximum value that can be represented by a `uint256` type, which is 2^256-1.

Impact:
An attacker could exploit this vulnerability to manipulate the return value of the `tokenURI()` function and potentially cause other problems in the contract.

Recommendation:
To fix this vulnerability, the `tokenURI()` function should be updated to check for integer overflow before using the `toString()` function. One way to do this would be to use the `assert()` function to check whether the value of `_tokenId` is less than or equal to the maximum value that can be represented by a `uint256` type. If the value exceeds this maximum, the `assert()` function should be used to raise an error and prevent the function from continuing.

# 2nd report for : Base.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol

## 1. Unchecked External Call in setBaseURI()

Summary:
The `setBaseURI()` function in the `Base` contract is vulnerable to an unchecked external call attack because it does not properly check the return value of the `_transferOwnership()` function before returning.

Impact:
An attacker could exploit this vulnerability by calling the `setBaseURI()` function and providing a malicious contract as the `_owner` argument. If the `_transferOwnership()` function call fails, the contract's state will be updated with the new `baseURI` value but the `setBaseURI()` function will still return `true`, indicating that the operation was successful. This could result in the contract becoming owned by an attacker and potentially allowing them to steal funds or take other malicious actions.

Recommendation:
To fix this vulnerability, the `setBaseURI()` function should be updated to properly check the return value of the `_transferOwnership()` function before returning. If the function call fails, the `setBaseURI()` function should also revert and not update the contract's state with the new `baseURI` value. This will prevent an attacker from potentially gaining control of the contract by exploiting this vulnerability.

## 2. Unprotected Setter Function in Base Contract

Summary:
The `setBaseURI()` function in the `Base` contract is vulnerable because it does not properly protect against unauthorized access. The function is marked as `external`, which means it can be called by any contract or account. However, the function has an `onlyOwner` modifier, which should prevent non-owners from calling the function.

Impact:
An attacker could exploit this vulnerability to set the `baseURI` variable to any value, potentially causing unintended behavior in the contract.

Recommendation:
To fix this vulnerability, the `setBaseURI()` function should be marked as `internal` instead of `external`. This will prevent contracts and accounts that are not part of the contract's inheritance chain from calling the function, which will ensure that only authorized code can call the function. Additionally, the `onlyOwner` modifier should be removed, as it is not necessary if the function is marked as `internal`.

## 3. Unrestricted Ownership Transfer in `initialize()`

Summary:
The `initialize()` function in the `Base` contract is vulnerable to unrestricted ownership transfer because it does not check the caller's address before transferring ownership of the contract to the caller.

Impact:
An attacker could exploit this vulnerability to gain ownership of the contract and potentially steal funds or perform other actions that are restricted to the owner of the contract.

Recommendation:
To fix this vulnerability, the `initialize()` function should be updated to check the caller's address before transferring ownership. This could be done by adding a `require()` statement that checks the caller's address against the expected owner address. For example, the function could be updated as follows:
```
function initialize(address _owner) public initializer {
    require(msg.sender == _owner, "Invalid owner address");
    _transferOwnership(_owner);
}
```
This will prevent any caller other than the expected owner from gaining ownership of the contract.

# 3rd report for : Generative.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol

## 1. Reentrancy Attack in `setScriptPiece()`

Summary:
The `setScriptPiece()` function in the `Generative` contract is vulnerable to a reentrancy attack because it does not properly check whether a contract call has completed before updating the internal state of the contract.

Impact:
An attacker could exploit this vulnerability to gain control of the contract's internal state and potentially steal funds from the contract.

Recommendation:
To fix this vulnerability, the `setScriptPiece()` function should be updated to use the `require()` function to check the return value of any contract calls made within the function. This will ensure that the contract call has completed before updating the internal state of the contract and will prevent reentrancy attacks.Additionally, the function should be updated to use the `bytes.length` property to check the length of the `scriptPieces[_id]` variable, rather than using the `bytes()` function, which can cause reentrancy attacks.

## 2. Unchecked External Function Call in tokenToSeed()

Summary:
The `tokenToSeed()` function in the `Generative` contract makes an external call to the `blockhash()` function without properly checking the return value. This can result in a call to an invalid address, which could potentially cause the contract to be permanently stuck in a bad state.

Impact:
An attacker could exploit this vulnerability to cause the `tokenToSeed()` function to call an invalid address, which could potentially result in the contract being stuck in an unrecoverable state. This could result in a loss of funds or other unintended behavior.

Recommendation:
To fix this vulnerability, the `tokenToSeed()` function should be updated to properly check the return value of the `blockhash()` function before using it. This could be done by checking that the returned value is not the zero address, or by using a more sophisticated method for checking the validity of the return value. Additionally, the function should be updated to handle the case where the `blockhash()` call fails, for example by throwing an error or returning a default value.

## 3. Unbounded Loop in setSeedBase()

Summary:
The `setSeedBase()` function in the `Generative` contract contains an unbounded loop that could potentially cause the contract to run out of gas. The loop is executed in the `keccak256()` call on the last line of the function, where the `abi.encodePacked()` function is used to encode an arbitrary number of input values.

Impact:
An attacker could exploit this vulnerability by calling the `setSeedBase()` function with a large number of input values, which would cause the contract to run out of gas and potentially become unresponsive.

Recommendation:
To fix this vulnerability, the `setSeedBase()` function should be updated to limit the number of input values that are passed to the `abi.encodePacked()` function. This could be done by using a fixed-sized array to store the input values and only passing the first n elements of the array to the `abi.encodePacked()` function, where n is the maximum number of input values that the function should accept. This will prevent the possibility of an unbounded loop and ensure that the contract can always complete the `setSeedBase()` function within the available gas limit.

# 4th report for : Escher721Factory.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol

## 1. Inadequate Access Control in createContract()

Summary:
The `createContract()` function in the `Escher721Factory` contract has inadequate access control because it only checks that the caller has the `CREATOR_ROLE`, but it does not check whether the caller has the appropriate permissions to create a new contract.

Impact:
An attacker could exploit this vulnerability to create new contracts without having the appropriate permissions. This could potentially result in unauthorized access to sensitive information or other unintended behavior.

Recommendation:
To fix this vulnerability, the `createContract()` function should be updated to check that the caller has the appropriate permissions to create a new contract. This could be done by adding a new access control modifier to the function that checks the caller's permissions, or by implementing a more sophisticated access control system. Additionally, the `initialize()` function should be updated to properly `initialize` the `implementation` and `uriClone` variables to safe default values.

## 2. Race Condition in createContract()

Summary:
The `createContract()` function in the `Escher721Factory` contract is vulnerable to a race condition because it uses the `clone()` function to create new contract instances, but it does not check whether the contract has been initialized before calling its `initialize()` function.

Impact:
An attacker could exploit this vulnerability to create multiple instances of the `Escher721` contract with the same address, which could potentially result in unintended behavior or loss of funds.

Recommendation:
To fix this vulnerability, the `createContract()` function should be updated to check whether the contract instance has been initialized before calling its `initialize()` function. This could be done by adding a new state variable to the `Escher721` contract that tracks whether it has been initialized, and checking this variable before calling `initialize()`. Additionally, the `createContract()` function should use a more secure method for creating new contract instances, such as the `create2` opcode, to prevent potential race conditions.

## 3. Unprotected Self-Destruct in createContract()

Summary:
The `createContract()` function in the `Escher721Factory` contract is vulnerable to an unprotected self-destruct attack because it does not check whether the contract being created is the contract that contains the `createContract()` function.

Impact:
An attacker could exploit this vulnerability to cause the contract to self-destruct and potentially steal funds from the contract.

Recommendation:
To fix this vulnerability, the `createContract()` function should be updated to check whether the contract being created is the `Escher721Factory` contract. If it is, the function should abort and not allow the contract to be created. This will prevent the contract from self-destructing and ensure the integrity of the contract's funds.

# 5th report for : FixedPriceFactory.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol

## 1. Inadequate Access Control in createFixedSale()

Summary:
The `createFixedSale()` function in the `FixedPriceFactory` contract has inadequate access control because it only checks that the caller has the `bytes32(0x00)` role, but it does not check whether the caller has the appropriate permissions to create a new contract.

Impact:
An attacker could exploit this vulnerability to create new contracts without having the appropriate permissions. This could potentially result in unauthorized access to sensitive information or other unintended behavior.

Recommendation:
To fix this vulnerability, the `createFixedSale()` function should be updated to check that the caller has the appropriate permissions to create a new contract. This could be done by adding a new access control modifier to the function that checks the caller's permissions, or by implementing a more sophisticated access control system. Additionally, the `initialize()` function should be updated to properly initialize the `implementation` variable to a safe default value.

## 2. Integer Overflow in createFixedSale()

Summary:
The `createFixedSale()` function in the `FixedPriceFactory` contract is vulnerable to an integer overflow attack because it does not properly check for the possibility of an overflow when comparing `_sale.finalId` and `_sale.currentId`.

Impact:
An attacker could exploit this vulnerability to cause an integer overflow, which could potentially result in a loss of funds or other unintended behavior.

Recommendation:
To fix this vulnerability, the `createFixedSale()` function should be updated to use a safe method for comparing `_sale.finalId` and `_sale.currentId`. One option would be to use the `SafeMath.gt()` function from the `openzeppelin-solidity` library, which will properly handle the possibility of an integer overflow. This will prevent the possibility of an integer overflow in the `createFixedSale()` function.

# 6th report for : OpenEditionFactory.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol

## 1. Inadequate Access Control and Integer Overflow in createOpenEdition()

Summary:
The `createOpenEdition()` function in the `OpenEditionFactory` contract has inadequate access control because it only checks that the caller has the `0x00` role, but it does not check whether the caller has the appropriate permissions to create a new contract. Additionally, the function is vulnerable to an integer overflow attack because it does not check that the `sale.endTime` value is strictly greater than the `sale.startTime value`. If the `sale.endTime` value is less than or equal to the `sale.startTime` value, the subtraction performed in the `require()` statement will cause an integer overflow and the condition will always evaluate to `true`.

Impact:
An attacker could exploit these vulnerabilities to create new contracts without having the appropriate permissions and cause an integer overflow, which could potentially result in a loss of funds or other unintended behavior.

Recommendation:
To fix these vulnerabilities, the `createOpenEdition()` function should be updated to check that the caller has the appropriate permissions to create a new contract and that the `sale.endTime` value is strictly greater than the `sale.startTime` value. This will prevent the possibility of an integer overflow and ensure the integrity of the contract's internal state. Additionally, the `initialize()` function should be updated to properly initialize the `implementation` variable to a safe default value.

## 2. Lack of Access Control on `setFeeReceiver` Function

Summary: 
The `setFeeReceiver` function in the `OpenEditionFactory` contract allows anyone to change the contract's fee receiver address, regardless of whether they are the owner of the contract or not.

Impact: 
This vulnerability allows an attacker to change the fee receiver address to their own address, allowing them to steal any fees collected by the contract.

Recommendation: 
The `setFeeReceiver` function should be restricted to only allow the contract owner to change the fee receiver address. This can be achieved by adding an access control modifier, such as `onlyOwner`, to the function definition.

# 7th report for : LPDAFactory.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol

## 1. Re-initialization of contract

Summary: 
The `initialize` function in the `LPDA` contract is called again during the `createLPDASale` function in the `LPDAFactory` contract. This could potentially allow the sale receiver and other sale parameters to be changed.

Impact: 
This vulnerability could allow an attacker to change the sale receiver and other sale parameters in an existing `LPDA` contract, potentially allowing them to steal funds.

Recommendation: 
The `initialize` function in the `LPDA` contract should be marked as `internal` or `private` to prevent it from being called again after the contract has been created. Alternatively, the `createLPDASale` function in the `LPDAFactory` contract could be modified to only set the sale receiver and other parameters in the `LPDA` contract if they have not been set yet.

## 2. Insufficient Input Validation for LPDA Sale Creation

Summary:
The `createLPDASale()` function in the `LPDAFactory` contract does not validate all input parameters provided in the `LPDA.Sale` struct. In particular, the `dropPerSecond` and `startPrice` parameters are not checked to ensure they are non-negative values.

Impact:
An attacker could create an LPDA sale with negative values for the `dropPerSecond` and `startPrice` parameters, potentially allowing them to gain a financial advantage by buying tokens at a lower price than intended.

Recommendation:
Add checks to the `createLPDASale()` function to ensure that the `dropPerSecond` and `startPrice` parameters are non-negative before creating the LPDA sale contract. This can be done by adding `require(sale.dropPerSecond >= 0, "INVALID DROP PER SECOND")` and `require(sale.startPrice >= 0, "INVALID START PRICE")` to the function.

# 8th report for : Escher.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol

## 1. Overriding ERC1155 `safeTransferFrom` and `safeBatchTransferFrom` functions

Summary: 
The `Escher` contract overrides the `safeTransferFrom` and `safeBatchTransferFrom` functions of the `ERC1155` contract, and makes them always revert with the error message "SoulBound". This prevents the contract from being used to transfer tokens.

Impact: 
This vulnerability prevents the contract from being used to transfer tokens, even in a safe and secure manner. This could prevent the contract from being used for its intended purpose, and could also make it difficult for users to interact with the contract.

Recommendation: 
The `safeTransferFrom` and `safeBatchTransferFrom` functions should be removed from the contract, or their implementation should be updated to allow for safe and secure token transfers. Alternatively, the contract could implement its own functions for transferring tokens in a safe and secure manner.

## 2. Improper Access Control in ERC1155

Summary: 
The `_grantRole` and `_revokeRole` functions in the `Escher` contract do not properly check for authorization before granting or revoking roles. Any address can grant or revoke roles by calling these functions, potentially allowing unauthorized access to restricted functions.

Impact: 
This vulnerability allows any address to grant or revoke roles, potentially allowing unauthorized access to restricted functions.

Recommendation: 
The `_grantRole` and `_revokeRole` functions should check for authorization before granting or revoking roles. The authorization check should ensure that only authorized addresses are able to grant or revoke roles. This can be achieved by checking that the caller has the appropriate role, or by adding a function that only authorized addresses can call to grant or revoke roles.

## 3. Invalid Interface ID

Summary: 
The `supportsInterface` function does not properly validate the input interface ID. As a result, it may return true for invalid interface IDs.

Impact: 
An attacker may be able to trick other contracts or users into thinking that the contract supports an interface that it does not actually support. This could lead to incorrect behavior and could potentially be exploited to cause loss of funds.

Recommendation: 
The `supportsInterface` function should validate the input interface ID against a list of known valid interface IDs before returning a value. This will ensure that the function only returns true for known, valid interfaces.

## 4. Reentrancy Vulnerability in Escher contract

Summary: 
The `_grantRole` and `_revokeRole` functions in the Escher contract can be vulnerable to reentrancy attacks, as they perform operations on external contracts (calling `_mint` and `_burn`) before checking the success of the operation.

Impact: 
An attacker can exploit this vulnerability to perform a reentrancy attack on the contract, potentially stealing funds or causing other malicious effects.

Recommendation: 
To fix this vulnerability, the contract should be refactored to check the success of the external contract operations before performing any other actions. This can be done by using the `require` statement to check for the success of the external contract operations, and reverting the transaction if they fail.

## 5. Using the `super` keyword

Summary: 
The `super` keyword is used in the `Escher` contract to call the `_grantRole` and `_revokeRole` functions of its parent contract `AccessControl`. This can cause problems if the parent contract is changed or upgraded, and the expected behavior of these functions changes.

Impact: 
If the parent contract is changed or upgraded, the `Escher` contract may not behave as intended. This can lead to incorrect grant or revocation of roles, and may result in loss of funds or other unintended consequences.

Recommendation: 
Instead of using the `super` keyword, the `Escher` contract should explicitly call the `_grantRole` and `_revokeRole` functions of the `AccessControl` contract. This will make it clear which contract's functions are being called, and will not be affected if the parent contract is changed or upgraded.

## 6. ERC1155 Integer Overflow

Summary:

The `_mint` and `_burn` functions in the `Escher` contract use the `uint256` type for the `_value` parameter, which is susceptible to integer overflow vulnerabilities.

Impact:

An attacker could exploit this vulnerability by providing a large value for the `_value` parameter, causing an integer overflow and resulting in unexpected behavior. This could lead to loss of funds, or allow the attacker to gain unauthorized access to certain functions in the contract.

Recommendation:

It is recommended to use the `SafeMath` library or similar to prevent integer overflows in the `_mint` and `_burn` functions.

## 7. Improper Role Granting

Summary: 
The `_grantRole()` function in the contract grants the `CREATOR_ROLE` to the specified `_account` without checking if the calling contract has the `CURATOR_ROLE`. This allows any contract to add creators to the system without permission.

Impact: 
This vulnerability allows any contract to add arbitrary accounts as creators without permission, potentially leading to loss of control over the marketplace.

Recommendation: 
The `_grantRole()` function should check if the calling contract has the `CURATOR_ROLE` before granting the `CREATOR_ROLE` to the specified `_account`. This will ensure that only authorized contracts can add creators to the system.

## 8. Unbounded loops in ERC1155

Summary: 
The `ERC1155` contract from the `OpenZeppelin` library has a potential vulnerability where the `_forEach` function can be called in an unbounded loop. This can result in a denial of service attack on the contract.

Impact: 
An attacker could potentially use this vulnerability to cause the contract to enter an unbounded loop, which would cause the contract to stop responding to requests. This could result in a denial of service attack on the contract.

Recommendation: 
This vulnerability can be mitigated by using the `require` statement to limit the number of iterations in the `_forEach` function. This would prevent the contract from entering an unbounded loop.

## 9. Permission Override Vulnerability in Escher Contract

Summary: 
The Escher contract allows anyone with the `DEFAULT_ADMIN_ROLE` to grant and revoke any role, including the `CURATOR_ROLE` and `CREATOR_ROLE`. This can be exploited by an attacker who gains access to the `DEFAULT_ADMIN_ROLE` to gain control over the contract and manipulate its functionality.

Impact: 
This vulnerability allows an attacker to gain control over the contract and manipulate its functionality, potentially leading to loss of funds or assets.

Recommendation: 
Restrict the `DEFAULT_ADMIN_ROLE` to only grant and revoke the `DEFAULT_ADMIN_ROLE` itself, and add additional roles and functions for granting and revoking other roles. This will limit the scope of the `DEFAULT_ADMIN_ROLE` and prevent it from being used to gain unauthorized access to the contract.

## 10. Reentrancy Vulnerability in Escher Contract

Summary: 
The `addCreator()` function in the Escher contract does not properly protect against reentrancy attacks. As a result, a malicious contract could call the `addCreator()` function and then immediately call `_grantRole()` before the `addCreator()` function has completed execution, potentially allowing the attacker to grant themselves additional roles.

Impact: 
If exploited, this vulnerability could allow an attacker to grant themselves additional roles and gain unauthorized access to sensitive functions in the contract.

Recommendation: 
The `addCreator()` function should be modified to use a mutex or other mechanism to protect against reentrancy attacks. Additionally, the `_grantRole()` and `_revokeRole()` functions should only be accessible to contract admins and not be callable by external contracts.

# 9th report for : Escher721.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol

## 1. Unrestricted minting of ERC721 tokens

Summary: 
The `mint` function in the contract is declared `public virtual` and is not restricted by any checks, allowing anyone to call it and mint ERC721 tokens.

Impact: 
This vulnerability allows any user to mint an unlimited number of ERC721 tokens, potentially leading to inflation and potentially diluting the value of existing tokens.

Recommendation: 
The `mint` function should be restricted to only allow execution by accounts with the `MINTER_ROLE`. This can be achieved by replacing `public virtual` with `onlyRole(MINTER_ROLE)` in the function declaration.

## 2. Unsafe Allow Constructor

Summary: 
The constructor of the `Escher721` contract is marked with the `unsafe-allow` modifier, which allows it to be called by anyone. This allows attackers to potentially execute arbitrary code in the contract's constructor.

Impact: 
Attackers could execute arbitrary code in the constructor, potentially allowing them to steal funds or manipulate the contract in other ways.

Recommendation: 
Remove the `unsafe-allow` modifier from the contract's constructor to prevent unauthorized access. Alternatively, the contract's constructor could be made external and only callable by authorized accounts.

## 3. TokenURI Delegate Vulnerability

Summary: 
The `tokenURI` function of the `Escher721` contract uses the `tokenUriDelegate` address to get the URI of the specified token ID. This can be exploited by an attacker who gains the `URI_SETTER_ROLE` and sets the `tokenUriDelegate` to a malicious contract that will return a false token URI. This can be used to mislead users about the nature of the token.

Impact: 
An attacker who can gain the `URI_SETTER_ROLE` can potentially trick users into buying tokens that are not what they appear to be, leading to financial loss for the users.

Recommendation: 
To fix this vulnerability, the `tokenURI` function should use a trusted URI source, such as a static IPFS hash, instead of relying on the `tokenUriDelegate` contract. The `tokenUriDelegate` contract should also be verified by a trusted party to ensure that it will return the correct URI for a given token ID. Alternatively, the `tokenURI` function could be removed altogether if it is not essential for the contract's functionality.

## 4. AccessControlUpgradeable.sol Reentrancy Vulnerability

Summary: 
The contract `Escher721` is vulnerable to reentrancy attacks through the `_grantRole()` and `_revokeRole()` functions, which are called in the `mint()` and `supportsInterface()` functions, respectively.

Impact: 
An attacker could exploit this vulnerability to execute arbitrary code in the context of the contract by calling `mint()` or `supportsInterface()` and calling one of the vulnerable functions in its fallback function. This could result in the attacker gaining unauthorized access to the contract's functions and potentially stealing funds.

Recommendation: 
The `mint()` and `supportsInterface()` functions should be modified to first check whether the caller has the required `MINTER_ROLE` or `DEFAULT_ADMIN_ROLE`, respectively, before calling `_grantRole()` or `_revokeRole()`. This will prevent the contract from being vulnerable to reentrancy attacks.

## 5. Unsafe Initialization

Summary: 
The `Escher721` contract has an `initialize` function that accepts a `_uri` parameter, but does not check if the provided `_uri` is valid and implements the required `ITokenUriDelegate` interface.

Impact: 
If an attacker provides an invalid `_uri` address, the `tokenURI` function will fail when called, potentially leading to contract failure or loss of funds.

Recommendation: 
Before setting the `tokenUriDelegate` address, the `initialize` function should check that the provided `_uri` address is a valid contract that implements the `ITokenUriDelegate` interface. This can be done by calling `require(address(new ITokenUriDelegate()).isImplementedBy(uri), "INVALID URI DELEGATE");`.

# 10th report for : FixedPrice.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol

## 1. Integer Overflow in the FixedPrice Contract

Summary: 
The `FixedPrice` contract contains a potential integer overflow vulnerability in the `buy` function. The `buy` function adds the value of the `_amount` parameter to the `currentId` field in the `Sale` struct. If the `_amount` parameter is large enough, it can cause the `currentId` field to overflow, allowing the attacker to buy more editions than the finalId value allows.

Impact: 
This vulnerability allows an attacker to buy more editions than the finalId value allows, potentially allowing them to sell the extra editions at a profit.

Recommendation: 
The `buy` function should check that the value of the `_amount` parameter, when added to the `currentId` field, does not exceed the `finalId` value. This can be done by replacing the `newId` variable with a check like the following:
```
require(sale_.currentId + _amount <= sale_.finalId, "TOO MANY");
```

## 2. Reentrancy vulnerability in contract FixedPrice

Summary:
The contract `FixedPrice` is vulnerable to a reentrancy attack. The vulnerability occurs in the `buy()` function, where the contract calls the `mint()` function of the NFT contract without checking for the return value. This allows an attacker to call the `buy()` function recursively and `mint` unlimited NFTs.

Impact:
An attacker can exploit this vulnerability to `mint` unlimited NFTs and drain the contract's funds.

Recommendation:
To prevent this vulnerability, the contract should check the return value of the `mint()` function before updating the `sale.currentId` variable.

## 3. `Buy()` Function Allows Purchase of More Tokens than Remaining Supply
Summary: 
The `buy()` function in the `FixedPrice` contract allows a user to buy more tokens than the remaining supply. This is because the `buy()` function checks if the new id of the token being bought is less than or equal to the final id, but it does not check if it is less than or equal to the remaining supply.

Impact: 
This vulnerability allows a user to buy more tokens than the remaining supply, potentially causing a loss of funds for the contract owner.

Recommendation: 
To fix this vulnerability, the `buy()` function should check if the new id of the token being bought is less than or equal to the remaining supply instead of the final id. This will prevent a user from buying more tokens than the remaining supply.

## 4. Unrestricted access to contract modification functions

Summary: 
The contract's `initialize()` and `cancel()` functions can be called by any user, allowing them to modify the contract's state.

Impact: 
This can potentially lead to unauthorized changes to the contract's state, such as altering the sale info or ending a sale prematurely.

Recommendation: 
Add access controls to the `initialize()` and `cancel()` functions, such as requiring the caller to be the contract owner or have a specific role.

## 5. Uninitialized Variables in FixedPrice Contract

Summary:
The FixedPrice contract has uninitialized variables in its constructor. This may lead to unpredictable behavior and potential vulnerabilities.

Impact:
Uninitialized variables may cause unexpected errors or vulnerabilities in the contract. This could result in loss of funds or unintended behavior.

Recommendation:
Initialize all variables in the contract's constructor to prevent unpredictable behavior and potential vulnerabilities.

## 6. Incorrect Type Used for Storage of Sale Prices

Summary:
The contract uses a uint96 type to store the sale price of an NFT, however, the maximum value of a uint96 is only 2^96-1. As the price of an NFT can potentially be much higher, using this type can cause overflow issues and result in incorrect prices being stored and used in the contract.

Impact:
This vulnerability can result in incorrect sale prices being stored and used in the contract, leading to incorrect calculations and potential loss of funds for users buying NFTs.

Recommendation:
It is recommended to use a larger integer type, such as uint128 or uint256, to store the sale price of an NFT in the contract to avoid overflow issues and ensure correct calculations.

## 7. Unsafe `_transferOwnership` function call

Summary:
The `_transferOwnership` function is called in the `initialize` function without any protection against reentrancy attacks. This could allow an attacker to call the `initialize` function again and transfer ownership of the contract multiple times.

Impact:
An attacker could transfer ownership of the contract multiple times, potentially leading to loss of control over the contract.

Recommendation:
To prevent reentrancy attacks, the `_transferOwnership` function should be called using the `transferOwnership` function from the `Ownable` contract, which includes a reentrancy guard.

# 11th report for: OpenEdition.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol

## 1. Incorrect Conversion of uint24 to uint256

Summary:
The `buy()` function in the contract `OpenEdition` contains a bug that can lead to incorrect values being stored and used. In this function, the `_amount` parameter is converted from type `uint256` to type `uint24`, but this conversion can lead to loss of precision if the value of `_amount` is greater than `2^24 - 1`.

Impact:

This vulnerability can lead to incorrect behavior in the contract, potentially allowing attackers to mint more NFTs than intended, or to mint NFTs with incorrect values. This can result in loss of funds for the contract and its users.

Recommendation:

To fix this issue, the `_amount` parameter should be converted to `uint256` before it is used in any calculations. This will ensure that the contract can handle values of `_amount` that are greater than `2^24 - 1` without loss of precision. Alternatively, the contract could be modified to only accept values of `_amount` that are less than or equal to `2^24 - 1`.

## 2. Reentrancy Vulnerability in OpenEdition Contract

Summary: 
The OpenEdition contract contains a reentrancy vulnerability in the `buy()` function. The vulnerability can be exploited by an attacker calling the `buy()` function and calling the `finalize()` function in the fallback function of the contract. This allows the attacker to repeatedly buy editions and receive a payout from the contract without increasing the sale's current id.

Impact: 
An attacker can exploit this vulnerability to repeatedly buy editions and receive a payout from the contract without increasing the sale's current id. This can result in a loss of funds for the contract and its users.

Recommendation: 
To fix this vulnerability, the `buy()` function should check that the contract is not in the process of being finalized before executing the sale. This can be done by adding a `bool` variable to track whether the contract is being finalized and checking its value in the `buy()` function.

## 3. OpenEdition contract allows unauthorized users to cancel a sale

Summary: 
The `cancel` function in the OpenEdition contract allows any user to cancel a sale, not just the owner. This can lead to a sale being cancelled without the owner's permission, resulting in financial loss.

Impact: 
This vulnerability can result in financial loss for the owner of the sale and can also cause market instability.

Recommendation: 
The `cancel` function should only be accessible to the contract owner, using the `onlyOwner` modifier. This can be implemented by adding the `onlyOwner` modifier to the function declaration, like so:

```
function cancel() external onlyOwner {
    require(block.timestamp < sale.startTime, "TOO LATE");
    _end(sale);
}
```

## 4. Lack of input validation for buy() function in OpenEdition contract

Summary: 
The buy() function in the OpenEdition contract does not validate the input provided by the msg.sender, which allows an attacker to buy unlimited amount of editions at the sale's fixed price.

Impact: 
An attacker can buy unlimited amount of editions at the sale's fixed price, causing potential financial loss to the contract owner and other users.

Recommendation: 
Validate the input provided by the msg.sender in the buy() function to ensure the amount of editions being bought is within the allowed limit.

## 5. Incorrect Use of initializer Function Modifier

Summary: 
The `OpenEdition` contract uses the `initializer` function modifier on the `initialize` function, but this modifier should not be used in this case. The `initialize` function is called directly in the `_createFixed` function in `ISaleFactory`, but the `initializer` modifier can only be called by the contract's creator or the contract's owner. As a result, any contract that implements `ISaleFactory` can initialize an `OpenEdition` contract, which could be exploited by malicious actors.

Impact: 
The incorrect use of the `initializer` function modifier in the `OpenEdition` contract could allow malicious actors to initialize the contract without the intended restrictions, potentially leading to loss of funds or other unintended behavior.

Recommendation: 
The `initialize` function in the `OpenEdition` contract should not use the `initializer` function modifier. Instead, the contract's creator or owner should call the `initialize` function directly. This will ensure that only the intended parties are able to initialize the `OpenEdition` contract.

## 6. ISale contract allows unauthorized address to mint an NFT.

Summary: 
The contract `OpenEdition` is missing a check to verify that the caller has the required permission to mint NFTs before calling the `mint` function of the `IEscher721` contract.

Impact: 
Attackers can mint NFTs without having the required permission, allowing them to create unauthorized editions of artworks.

Recommendation: 
Add a check to verify that the caller has the required permission before calling the `mint` function of the `IEscher721` contract. For example, by checking the `hasRole` function of the `IEscher721` contract.

## 7. Unbounded Loop Vulnerability in OpenEdition Contract

Summary:

The `OpenEdition` contract contains a vulnerability in the `buy` function that allows an attacker to cause an unbounded loop. The loop in question is the `for `loop at line 50, where the stopping condition is `x <= newId`. However, `newId` is calculated as `amount + temp.currentId` where `amount` is a `uint24`, while `temp.currentId` is also a `uint24`. The result of adding two `uint24s` is not guaranteed to be within the range of `uint24`, which means that the loop may never stop if `amount + temp.currentId` overflows. This can be exploited by an attacker to cause a denial of service (DoS) attack on the contract by calling the buy function with a large enough value for `amount`.

Impact:

An attacker can cause a denial of service (DoS) attack on the contract by calling the buy function with a large enough value for `amount`. This would effectively make the contract unusable for the duration of the attack.

Recommendation:

The recommended fix for this vulnerability is to ensure that the result of adding two `uint24s` is within the range of `uint24` by explicitly casting one of the operands to `uint256` before adding them. For example, the code can be changed to:
```
uint24 newId = uint24(uint256(amount) + temp.currentId);
```

This will ensure that the result of adding two `uint24s` is within the range of `uint24`, and the loop will stop as expected.

# 12th report for: LPDA.sol https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

## 1. Incorrect value comparison in LPDA contract

Summary: 
The contract has a bug in the `buy` function. The `msg.value` and `amount * price` comparison is checking if `msg.value` is greater than or equal to `amount * price`, but it should be checking if they are equal.

Impact: 
This bug could allow users to buy the edition at a lower price than intended.

Recommendation: 
The comparison in the `buy` function should be changed to use the `==` operator instead of `>=`.

## 2. Refund Function Incorrectly Calculates Owed Amount

Summary: 
The `refund` function in the `LPDA` contract calculates the amount owed to the caller incorrectly. The `owed` variable is calculated by subtracting the current price of the items bought from the caller's balance, rather than the original price of the items. This means that the caller is refunded less than they are entitled to.

Impact: 
This bug allows the contract owner to keep a portion of the funds that should be refunded to the caller.

Recommendation: 
The `owed` variable should be calculated by subtracting the original price of the items bought from the caller's balance. The line `uint80 price = uint80(getPrice()) * r.amount` should be replaced with `uint80 price = uint80(sale.startPrice) * r.amount`.

## 3. Possible Divide By Zero Error in `getPrice()` Function

Summary: 
The `getPrice()` function in the contract has a potential divide by zero error. If `dropPerSecond` is 0, then the code will try to divide by 0, which will throw an error and cause the contract to fail.

Impact: 
If this error is triggered, it could cause the contract to stop functioning and prevent users from buying editions. This could result in lost funds for users who have already purchased editions.

Recommendation: 
Update the code to check for a divide by zero error and handle it gracefully. For example, you could set `dropPerSecond` to a minimum non-zero value if it is currently 0. Alternatively, you could add a condition to the code to only perform the division if `dropPerSecond` is non-zero.

## 4. Lack of Validation of Inputs in "buy" Function

Summary:
The "buy" function in the LPDA contract allows users to purchase items by calling the function and passing in the amount they want to buy as an argument. However, there is no validation of this input, meaning that an attacker could call the function with a value that is larger than the maximum amount allowed. This can cause the amountSold variable to be set to a value larger than the finalId, causing an error when the final price is calculated.

Impact:
If an attacker is able to call the "buy" function with an invalid amount, they could cause the final price calculation to fail. This could result in the contract being unable to pay out the correct amount to the sale receiver and the factory fee receiver.

Recommendation:
To prevent this vulnerability, the "buy" function should validate the input amount to ensure that it does not exceed the maximum allowed value. This can be done by adding a check to the function that verifies that the newId calculated from the input amount is less than or equal to the finalId.

## 5. Potential overflow in 'LPDA' contract

Summary:
In the 'LPDA' contract, there is a potential for overflow when using the 'uint48' and 'uint80' types to store values in the 'amountSold' and 'Receipt.balance' variables. These variables may potentially be assigned values larger than their maximum capacity, causing the contract to behave unexpectedly.

Impact:
If an attacker is able to manipulate the values stored in these variables, they may be able to cause the contract to behave unexpectedly. This could potentially result in the attacker being able to gain an unfair advantage over other users of the contract.

Recommendation:
To avoid this potential overflow issue, it is recommended to use the 'uint256' type for the 'amountSold' and 'Receipt.balance' variables. This will ensure that these variables have enough capacity to store any value that may be assigned to them, and prevent the potential for overflow.

## 6. Sale Receiver Cannot Refund Their Own Purchases

Summary: 
In the `refund` function, the `saleReceiver` address is able to refund their own purchases, which should not be allowed. This is because the `saleReceiver` address is checked against the `msg.sender` address instead of the address of the `Receipt` being refunded.

Impact: 
The `saleReceiver` address can refund their own purchases, potentially allowing them to receive more funds than they should.

Recommendation: 
In the `refund` function, check the `msg.sender` address against the address associated with the `Receipt` being refunded instead of the `saleReceiver` address. This will prevent the `saleReceiver` address from refunding their own purchases.

## 7. Potential Integer Overflow in `buy()` Function

Summary:
The `buy()` function in the `LPDA` contract has a potential integer overflow vulnerability in the `newId` variable. The `newId` variable is calculated by adding the `amount` variable to the `temp.currentId` variable. If the `amount` variable is large enough, the sum of these two variables could exceed the maximum value that `newId` can hold, resulting in an integer overflow.

Impact:
If an attacker is able to successfully exploit this vulnerability, they could potentially buy a large number of editions at a very low price. This could result in significant financial losses for the contract owner and other users of the contract.

Recommendation:
To address this vulnerability, the `newId` variable should be calculated using the `SafeMath` library to prevent integer overflows. The `require()` statement that checks the maximum number of editions that can be bought should also be moved to after the `newId` variable is calculated, to ensure that it is not possible to exceed this limit.