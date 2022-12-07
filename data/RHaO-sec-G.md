
Use custom errors rather than revert()/require() strings to save deployment gas.
 gas optimization issue found on solidity files
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol (Various lines throughout the file )
2.https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol (Various lines throughout the file )
3.https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol (Various lines throughout the file )
4.https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol (Various lines throughout the file )
5.https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol (Various lines throughout the file )
6.https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol (Various lines throughout the file )
7.https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol (Various lines throughout the file )


