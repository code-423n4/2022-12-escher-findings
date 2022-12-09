
LOW 

[1] Floating pragma should not be used in production
    -It is considered best practice to pick one complier version and stick with it. with a floating pragma, contracts mauy accidentally be deployed using an outdated or problematic compiler version which can cause bugs. For more information read here https://swcregistry.io/docs/SWC-103

        There are 12 instances of this issue:

Generative
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L2

Unique
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol#L2

Base
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L2

Escher721
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L2

Escher721Factory
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol#L2

Escher
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L2

FixedPriceFactory
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L2

OpenEditionFactory
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L2

LPDAFactory
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L2

FixedPrice
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L2

OpenEdition
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L2

LPDA
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L2


[2] Project should have complete documentation for better readability. 
    - most functions have documentation explaining what the function does and the purpose of each parameter. However there are a few functions with no information beyond the code itself. For better readability it is reccomoended that all functions have consistant documentation.

        there are 11 instances of this issue:

Generative
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L19
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L27

Unique
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol#L10

Base
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L10
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L14
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L18

Escher721
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L84


OpenEdition
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L116
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L120


LPDA
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L142
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L146

[3] Place revert inside _burn method to ensure funcationality cannot be utilised. 
    - It appears that burn is not intened to be be used by anyone in the project. Burn is not apart of the specs from what I saw and there seems to be no access point for users or owners to burn. With that in mind I would still reccomend placing a revert inside of this function to prevent any unexcpexted attacks or access. 

    there is 1 instance of this issue:

Escher721
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L97

[4] Name mixup in documentation
    - both in openeditionFacotry and LPDAFactory the setFeeReciever function refers to the puppose of the function as setting the fee reciever for 'fixed price editons' when what the function is actually doing is settin gthe fee reciever for open edition and LPDA. I would recomend the the comment get changed to 
    /// @notice set the fee receiver for Open Edition  editions 
    and
    /// @notice set the fee receiver for LPDA editions 

    there are 2 instance of this issue:

OpenEditionFactory
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L40

LPDAFactory
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L44

[5] Wrong variable names cited in documentation. 
    - In fixedPrice One of the comments says, "store nextId and remainingSupply, where nextId increases and remainingSupply decreases to 0". Neither of these variables are in this current project. For more readability and less confusion it is reccomend that this comment is either removed or updated with the correct varaibles. 

    there is 1 instance of this issue:

FixedPrice
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L12

[6] Inconsistnat formatting when initiating sale in the constructor. 
    - in FixedPrice, Openedition, and LPDA the sale object is formatted differently in every single one. 
        - in fixed price values are directly being passed into the object. And the code is all in one line 
        - in open edition the values are also directly being passed into the object. however, the code is broken up into multiple lines. 
        - lastly in LPDA variables are created prior to the sale object being created and then the variables are being passed through. The sale object is created on one line. 
    - None of these ways are nessisarily wrong. However when essentialy the same thing is done in three differnt ways it makes the readability hard and the consistantcy less than optimal. 
    - recomendation is to pick one way and use that for all three contracts. 

    Instances of these issues are below 


FixedPrice
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L46

OpenEdition
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L45

LPDA
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L53


[7] Inconsistnat naming of variables
    - in FixedPrice in the buy() function you create a local version of the sale object called "sale" 
    - in openEdition and LPDA in the buy() fucntion you create a local veriosn of the sale object called "temp" 
    - both variables serve more or less the same purpose within there respective functions. For readability and consistancy I would recomend that the local variable either be called just "sale" or just "temp" in all three contracts. Not a combination of both. 

    Instances of these issues are below 

FixedPrice
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L58

OpenEdition
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L59
