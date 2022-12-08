### 1st Bug
Cache Array Length Outside of Loop

#### Impact
Issue Information: 
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Example
🤦 Bad:

for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
🚀 Good:

uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}
#### Findings:
```
2022-12-escher/src/uris/Generative.sol::15 => require(bytes(scriptPieces[_id]).length == 0, "ALREADY SET");
```
#### Tools used
Manual




### 2nd Bug
Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: 
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.

Example
🤦 Bad:

bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
🚀 Good:

bytes32 public immutable GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");

#### Findings:
```
2022-12-escher/src/Escher.sol::11 => bytes32 public constant CREATOR_ROLE = keccak256("CREATOR_ROLE");
2022-12-escher/src/Escher.sol::13 => bytes32 public constant CURATOR_ROLE = keccak256("CURATOR_ROLE");
2022-12-escher/src/Escher721.sol::17 => bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");
2022-12-escher/src/Escher721.sol::19 => bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
2022-12-escher/src/uris/Generative.sol::24 => seedBase = keccak256(abi.encodePacked(numb, blockhash(numb - 1), time, (time % 200) + 1));
2022-12-escher/src/uris/Generative.sol::29 => return keccak256(abi.encodePacked(_tokenId, seedBase));
```
#### Tools used
Manual




### 3rd Bug
Long Revert Strings

#### Impact
Issue Information: 
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

Example
🤦 Bad:

require(condition, "UniswapV3: The reentrancy guard. A transaction cannot re-enter the pool mid-swap");
🚀 Good (with shorter string):

// TODO: Provide link to a reference of error codes
require(condition, "LOK");
🚀 Good (with custom errors):

/// @notice A transaction cannot re-enter the pool mid-swap.
error NoReentrancy();

// ...

if (!condition) {
    revert NoReentrancy();
}

#### Findings:
```
2022-12-escher/src/Escher.sol::4 => import {ERC1155} from "openzeppelin/token/ERC1155/ERC1155.sol";
2022-12-escher/src/Escher.sol::5 => import {AccessControl} from "openzeppelin/access/AccessControl.sol";
2022-12-escher/src/Escher721.sol::4 => import {ITokenUriDelegate} from "./interfaces/ITokenUriDelegate.sol";
2022-12-escher/src/Escher721.sol::5 => import {Initializable} from "openzeppelin-upgradeable/proxy/utils/Initializable.sol";
2022-12-escher/src/Escher721.sol::6 => import {ERC721Upgradeable} from "openzeppelin-upgradeable/token/ERC721/ERC721Upgradeable.sol";
2022-12-escher/src/Escher721.sol::7 => import {ERC2981Upgradeable} from "openzeppelin-upgradeable/token/common/ERC2981Upgradeable.sol";
2022-12-escher/src/Escher721.sol::8 => import {AccessControlUpgradeable} from "openzeppelin-upgradeable/access/AccessControlUpgradeable.sol";
2022-12-escher/src/Escher721Factory.sol::7 => import {ITokenUriDelegate} from "./interfaces/ITokenUriDelegate.sol";
2022-12-escher/src/minters/FixedPrice.sol::7 => import {Initializable} from "openzeppelin-upgradeable/proxy/utils/Initializable.sol";
2022-12-escher/src/minters/FixedPrice.sol::8 => import {OwnableUpgradeable} from "openzeppelin-upgradeable/access/OwnableUpgradeable.sol";
2022-12-escher/src/minters/LPDA.sol::7 => import {Initializable} from "openzeppelin-upgradeable/proxy/utils/Initializable.sol";
2022-12-escher/src/minters/LPDA.sol::8 => import {OwnableUpgradeable} from "openzeppelin-upgradeable/access/OwnableUpgradeable.sol";
2022-12-escher/src/minters/OpenEdition.sol::7 => import {Initializable} from "openzeppelin-upgradeable/proxy/utils/Initializable.sol";
2022-12-escher/src/minters/OpenEdition.sol::8 => import {OwnableUpgradeable} from "openzeppelin-upgradeable/access/OwnableUpgradeable.sol";
2022-12-escher/src/uris/Base.sol::4 => import {ITokenUriDelegate} from "../interfaces/ITokenUriDelegate.sol";
2022-12-escher/src/uris/Base.sol::5 => import {OwnableUpgradeable} from "openzeppelin-upgradeable/access/OwnableUpgradeable.sol";
```
#### Tools used
Manual