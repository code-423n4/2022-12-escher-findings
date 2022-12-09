# Report
## Non-Critical Issues ##
### [N-1]: Change type of function parameter  
**Context:**  
```
src\minters\LPDA.sol:
  59:         uint48 amount = uint48(_amount);
```
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#59

```
src\minters\OpenEdition.sol:
  58:         uint24 amount = uint24(_amount);
```
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#58

**Description:**  
You can specify the desired type for the function parameter, and then there is no need to do type casting.

### [N-2]: NatSpec is missing
**Context:**

1. https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L31
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L84
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L142
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L146
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L116
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L120
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L19
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L27
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol
 
### [N-3]: Use immutable instead of constant with keccak expression  
**Context:**  
```
src\Escher.sol:
  10      /// @notice The role that grants the ability to mint editions
  11:     bytes32 public constant CREATOR_ROLE = keccak256("CREATOR_ROLE"); 
  12      /// @notice the role that grants the ability to onboard creators
  13:     bytes32 public constant CURATOR_ROLE = keccak256("CURATOR_ROLE"); 
```
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#11

```
src\Escher721.sol:
  16      /// @notice The role to update the URI delegate
  17:     bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");  
  18      /// @notice The role to mint editionized art work
  19:     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");  
```
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#17

**Description:**  
According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/contracts.html#constant-and-immutable-state-variables) for a constant variable, the expression assigned to it is copied to all the places where it is accessed and also re-evaluated each time. It is recommended to use immutable instead. 