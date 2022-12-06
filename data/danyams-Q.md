# Struct Definition Should Not Come After State Variable Declaration

The struct Sale in FixedPrice.sol should come before the declaration of the state variable, factory

# FixedPrice.sol, LPDA.sol, and OpenEdition.sol do not need to be a Proxies


# SetFeeReceiver( ) in Factor Contracts can be made External

SetFeeReceiver( ) in FixedPriceFactory.sol, LPDAFactory.sol, and OpenEditionFactory.sol is never called internally.  Therefore, the function can be made external