Avoid leaving a contract uninitialized.

An uninitialized contract can be taken over by an attacker. This applies to both a proxy and its implementation
contract, which may impact the proxy. To prevent the implementation contract from being used, you should invoke
the {_disableInitializers} function in the constructor to automatically lock it when it is deployed:

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L25