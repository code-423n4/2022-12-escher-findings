Gas

[1] Mark onlyOwner functions as payable
    - Becasue the function will revert when called by anyone besides the owner due to modifiers such as onlyowner. 
gas can be saved by marking those functions as payable.
Marking the function as payable will lower the gas cost for legitimate callers 
because the compiler will not include checks for whether a payment was provided.
This will save cost whenever the function is called, as well as during deployment.
Base 

There are 4 instances of this issue:

Generative
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L19

FixedPriceFactory
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42

OpenEditionFactory
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42


LPDAFactory
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46

[2] When Incrementing in a for loop uncheck each increment. 
    - Inside of a for loop the variable being incremented cannot overflow because the loop checks each iteration to see if that variable is less than or equal to an specific amount. 
https://gist.github.com/devNamedKiki/d63ee50856c8ff1861d5412fecf8b2fb
There are 3 instances of this issue:

FixedPrice
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65

LPDA
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73

OpenEdition
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66

[3] x += y cost more gas. You can use x = x + y instead.
    - Although it is easier to change the value of a variable by using += or -=. It cost more gas than the written out version x = x + y and x = x - y.

There are 3 instances of this issue:

LPDA
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L66
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L70
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L71
