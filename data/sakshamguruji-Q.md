## POSSIBILITY TO FRONTRUN THE INITIALIZE FUNCTION

### Description:

The initialize function here https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L78 (and other initialize functions)
can be frontrun , i.e attacker can call the initialize function prior to the team and deploy the contract with his/her value.
Consider adding access modifier to this function.

## MISLEADING COMMENT 

### Description:

The comment here https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L27 says it is creating
a fixed sale proxy when it should be saying that it creates open edition proxy. This was probably copy pasted from the Fixed Proxy contract.
Similar error here https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L91

## SETTING OF PRIVILEGED ROLE SHOULD BE A TWO STEP PROCESS

### Description:

The setter function where we are assigning roles the executor and relay roles should be converted to a 2 step process , so that in a scenario
while setting the role if by mistake the owner sets the wrong address or some invalid address , instead of redeploying the whole contract we can
set the `tempUriDelegate`(example) and then the `tempUriDelegate` accepts his/her role and then we assign the tokenUriDelegate as the delegate.

Convert this function into a 2-step process -
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L57

## EXPRESSIONS FOR CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

### Description:

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use
the right tool for the task at hand. There is a difference between constant variables and immutable variables, and they
should each be used in their appropriate contexts. constants should be used for literal values written into the code, and
immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

Affected Instances:

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L17
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L19
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L11
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L13

## ZERO-ADDRESS CHECKS ARE MISSING

### Description:

Zero-address checks are a best practice for input validation of critical address parameters. While the codebase applies this to most cases, there are many places where this is missing in constructors and setters.
Impact: Accidental use of zero-addresses may result in exceptions, burn fees/tokens, or force redeployment of contracts.

Affected Instances:
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L29
In this function there is no check to see if the  `saleReceiver` is a 0 address.
Similarly here  https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L27
More instances:

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L51 (`to` address)
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L57
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L65
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L33
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L34
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L27
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L60
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L67

## NAME THE VARIABLE _tokenUriDelegate INSTEAD OF _uri

### Description:

The general convention is to name the parameter variables  in a constructor with an underscore  Ex - If there is a global variable `var`
and we are setting this variable inside the constructor , then the parameter would be named `_var` to make the code understandable what variable we are setting.

Change the parameter name here https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L34 to `_tokenUriDelegate` instead.

## FUNCTIONS LACK COMMENTS

### Description:

It is very important to have natspec comments for functions to help the reader understand the functionality of the function . It is always a good practice to have detailed comments/explanation for the function.

Affected instances:
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L84
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L12-L13
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L116
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L125

## EVENT IS MISSING INDEXED FIELDS

### Description:

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index
field costs extra gas during emission, so it’s not necessarily best to index the maximum allowed per event (three fields).
Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for
the events in question. If there are fewer than three fields, all of the fields should be indexed.

Affected instances:
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L31
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L34
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L40
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L30
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L33
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L39
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L41
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L42
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L43


## PUBLIC FUNCTIONS NOT CALLED WITHIN THE CONTRACT SHOULD BE MARKED AS EXTERNAL

### Description:

The functions that are not being called within the contract should be marked as external instead to adhere to the convention.


https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L32
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L51
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L57
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L64
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L72
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L78
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L84
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L78
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L86
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L91
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L96
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L101
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L21
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L27
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L31
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L38
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L49
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L82
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L89
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L97
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L102
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L107
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L112
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L116
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L99
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L110
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L117
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L128
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L133
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L138
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L142
