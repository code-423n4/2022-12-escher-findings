# Struct Definition Should Not Come After State Variable Declaration

The struct Sale in FixedPrice.sol should come before the declaration of the state variable, factory

# FixedPrice.sol, LPDA.sol, and OpenEdition.sol do not need to be a Proxies


# SetFeeReceiver( ) in Factor Contracts can be made External

SetFeeReceiver( ) in FixedPriceFactory.sol, LPDAFactory.sol, and OpenEditionFactory.sol is never called internally.  Therefore, the function can be made external

# Open Editions cannot be sold for more than 4722 ether 

OpenEdition.sol has a struct Sale, that contains $\2^72 for its member variable, price.  $\2^72 wei is roughly 4722 ether, meaning that this is the max limit that Open Edition NFTs can be sold for

