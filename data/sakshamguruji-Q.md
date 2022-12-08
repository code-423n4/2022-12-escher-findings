## MISLEADING COMMENT 

### Description:

The comment here https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L27 says it is creating
a fixed sale proxy when it should be saying that it creates open edition proxy. This was probably copy pasted from the Fixed Proxy contract.

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


