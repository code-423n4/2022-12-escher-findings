1- Using calldata instead of memory reduces gas cost:
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L35-L36

2- Using custom errors instead of require statements reduces gas: (for example:)
https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/OpenEdition.sol#L61-L62
https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/OpenEditionFactory.sol#L30-L32 

3- Using ```++i``` instead of ```i++``` is better in for loops