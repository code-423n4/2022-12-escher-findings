Presence of unused variables

The function "tokenURI" at https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol takes a uint256 as argument but it does not process it at all, as it will always returns the baseURI anyways (whatever the number is)

Unused variables are allowed in Solidity and they do not pose a direct security issue. It is best practice though to avoid them as they can:

cause an increase in computations (and unnecessary gas consumption)
indicate bugs or malformed data structures and they are generally a sign of poor code quality
cause code noise and decrease readability of the code

source : SWC-131