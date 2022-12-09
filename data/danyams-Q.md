# [01] Buyers of LPDA risk not being refunded

If an LPDA sale has greater demand than expected, a user may never be refunded.  This occurs since the balance of the contract will be transferred out in the **buy( )** function once the sale's **final id** is reached.  

**Recommended Mitigation Steps:**

A solution would be to separate the transfers into a separate function and make it callable only after the sale's **end time** has been reached.

Referenced Code: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L81-L88

# [02] Open Editions cannot be sold for more than 4722 ether 

OpenEdition.sol has a struct **Sale**, that contains 2^72  for its member variable, **price**.  2^72  wei is roughly 4722 ether, meaning that this is the max limit that an Open Edition NFT can be sold for.

**Recommended Mitigation Steps:**

Change price to a uint80, which would lead to a max price of 1208925 ether.

Referenced Code: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L15

# [03] Sales and Escher721 all use **mint( )** instead of **safeMint( )**

Using **safeMint( )** rather than **mint( )** leads to better composability and expected behavior when the caller is another contract.

# [04] Transfer function is used to transfer ether 

It is better to use **call( )** to send Ether since gas costs of opcodes are susceptible to change.

# [05] block.timestamp and block.number are used for entropy

block.timestamp and block.number are used for entropy in Generative.sol.  This will lead to two Generative contracts that both call setSeedBase() in the same block to have the same seedbase.  An oracle is a more reliable source of entropy.

Referenced Code: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L22-L25

# [06] uints are not safely casted 

All casting of uints should use a safe cast library, such as Solmate's SafeCastLib.  Not doing so leads to strange behavior when **buy( )** is called in a **FixedPrice** sale.  

A function parameter, **_amount**, is a uint256.  **_amount** is later casted to a uint48, but this casted value is not used when emitting an event.  This means that 2^48  can be passed in as an argument, which will later cast to 0 and no NFTs being purchased.  However, an event will be emitted that 2^48 NFTs were bought.

Referenced Code: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L57-L71

# [07] Public functions that are not used internally should use the visibility modifier **external**

Numerous functions throughout the codebase have a visibility modifier of **public** when they should use **external**

# [08] Reference types that are used as function arguments should use **calldata** instead of **memory**

There are many reference types that are passed in as function arguments do not change in value.  They should be marked as **calldata** in place of **memory**.  Specifically, this occurs frequently when a struct of type **Sale** is passed in as a function argument 

# [09] Unnecessary Inheritance

OpenEdition, FixedPrice, LPDA do not need to inherit Initializable since it is already inherited in OwnableUpgradable.  Escher721 also does not need to inherit Initilizable since it is already inherited in ERC721Upgradable.  

Referenced Code: 

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/25aabd286e002a1526c345c8db259d57bdf0ad28/contracts/token/ERC721/ERC721Upgradeable.sol#L20

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/25aabd286e002a1526c345c8db259d57bdf0ad28/contracts/access/OwnableUpgradeable.sol#L21

# [10] Function has visibility modifier **internal** but is not used internally

**tokenToSeed( )** in Generative.sol has visibility modifier **internal** but is not used internally

# [11] Return values are not used

The return values in **createOpenEdition( )**, **createLPDASale( )**, and **createFixedSale( )**, and **createContract( )** are not used.  If the return value is meant to be used to keep track of the new clone's address, that information can be found in the emitted event.

Referenced Code: 

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L31
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L29
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L29
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L29



