##

## [NC -1]  EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand. There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts. constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

> instances (4) 

[File: 2022-12-escher/src/Escher.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol)

     11:  bytes32 public constant CREATOR_ROLE = keccak256("CREATOR_ROLE");

    13:   bytes32 public constant CURATOR_ROLE = keccak256("CURATOR_ROLE");

[File: 2022-12-escher/src/Escher721.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol)

    17:    bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");
    
    19:    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

##

##  [NC -2] CONSTANT REDEFINED ELSEWHERE

Consider defining in only one contract so that values cannot become out of sync when only one location is updated. A cheap way to store constants in a single location is to create an internal constant in a library. If the variable is a local cache of another contract’s value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don’t get out of sync.

##

## [NC-3]  NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

> instances (4) 

[File: 2022-12-escher/src/interfaces/IEscher721.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/interfaces/IEscher721.sol)

[File: 2022-12-escher/src/interfaces/ISale.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/interfaces/ISale.sol)

[File: 2022-12-escher/src/interfaces/ISaleFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/interfaces/ISaleFactory.sol)

[File: 2022-12-escher/src/interfaces/ITokenUriDelegate.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/interfaces/ITokenUriDelegate.sol)

        pragma solidity ^0.8.17;

##

## [NC-4]  It is bad practice to use numbers directly in code without explanation

[File: 2022-12-escher/src/minters/LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

     84:   uint256 fee = totalSale / 20; // 20 used without explanations 

##

## [NC-5]  PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

##

     










