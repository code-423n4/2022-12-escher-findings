# Struct Definition Should Not Come After State Variable Declaration

The struct Sale in FixedPrice.sol should come before the declaration of the state variable, factory

# FixedPrice.sol, LPDA.sol, and OpenEdition.sol do not need to be a Proxies


# SetFeeReceiver( ) in Factor Contracts can be made External

SetFeeReceiver( ) in FixedPriceFactory.sol, LPDAFactory.sol, and OpenEditionFactory.sol is never called internally.  Therefore, the function can be made external

## Open Editions cannot be sold for more than 4722 ether 

OpenEdition.sol has a struct Sale, that contains $\2^72 for its member variable, price.  $\2^72 wei is roughly 4722 ether, meaning that this is the max limit that an Open Edition NFT can be sold for

# XXX, XXX, XXX all use mint instead of SafeMint( ) 

# block.timestamp and block.number Are Used For Entropy

block.timestamp and block.number are used for entropy in Generative.sol.  However, block.number is not random, while block.timestamp can be manipulated by miners.  Moreover, two Generative contracts, which both call setSeedBase() in the same block will have the same seedbase.  



Lines 26 - 28 in Generative.sol

