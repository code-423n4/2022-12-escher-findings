1- The function createContract() misses the return address value. There is just an emit in which the applications that subscribed to the event or the listener functions in the front-end get awarded of existence of such addresses:
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol#L31

2- It's better to use greater than instead of a 'strict-equality' for the require statement in the buy function in the FixedPrice.sol:
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L61

3- Escher721 token inherits from OpenZeppelin's upgradeable contracts however there is no upgradeability mechanism in the context. In this manner the storage layout changes with the presence of the gaps.
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol