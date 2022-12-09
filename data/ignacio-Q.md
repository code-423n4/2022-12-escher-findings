# The initialize function that initializes important contract state can be called by anyone.

Occurences:
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L18 
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L110

The attacker can initialize the contract before the legitimate deployer, hoping that the victim continues to use the same contract. In the best case for the victim, they notice it and have to redeploy their contract costing gas.

Recommend using the constructor to initialize non-proxied contracts. For initializing proxy contracts, recommend deploying contracts using a factory contract that immediately calls initialize after deployment, or make sure to call it immediately after deployment and verify the transaction

# use of   selfdestruc ()
Consider removing the self-destruct functionality unless it is absolutely required. If there is a valid use-case, it is recommended to implement a multisig scheme so that multiple parties must approve the self-destruct action.
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L122
#  ABI.ENCODEPACKED() SHOULD NOT BE USED WITH DYNAMIC TYPES WHEN PASSING THE RESULT TO A HASH FUNCTION SUCH AS KECCAK256()


If you use keccak256(abi.encodePacked(a, b)) and both a and b are dynamic types, it is easy to craft collisions in the hash value by moving parts of a into b and vice-versa. More specifically, abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c").

two or more dynamic types are passed to abi.encodePacked. Moreover, these dynamic values are user-specified function arguments in external functions, meaning anyone can directly specify the value of these arguments when calling the function. The signedOnly modifier is supposed to protect functions to permit only function arguments that have been properly signed to be passed to the function logic, but because a collision can be created, the modifier can be bypassed for certain select inputs that result in the same encodePacked value.

Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead

https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L24