1.Use of constant keccak variables results in extra hashing (and so gas).(4 instances)

This results in the keccak operation being performed whenever the variable is used, increasing gas costs relative to just storing the output hash. Changing to immutable will only perform hashing on contract deployment which will save gas.

code snippet:-
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L11
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L13
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L17
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L19

Recommended Mitigation Steps:-

Change the variable to be immutable rather than constant

2. use x=x+y instead of x+=y:-
Using the addition operator instead of plus-equals saves 113 gas.

code snippet:-
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L66
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L70
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L71


3. Increments/decrements and expressions  can be unchecked :-
In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

 have a look :-https://github.com/ethereum/solidity/issues/10695

Consider wrapping with an unchecked block here (around 25 gas saved per instance):

code snippet:-
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L62
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L64
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L67

4.Reduce the size of error messages (Long revert Strings):-
Shortening revert strings to fit in 32 bytes will decrease deployment time gas and will decrease runtime gas when the revert condition is met.
Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

code snippet:-
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L36
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L35
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L32
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L31
https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L28

5.storage pointer to a structure is cheaper than copying each value of the structure into memory, same for array and mapping:-

It may not be obvious, but every time you copy a storage struct/array/mapping to a memory variable, you are copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper. Exception: case when you need to read all or many members multiple times. In report included only cases that saved gas

code snipppet:-
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L60
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L100
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L118
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L59
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L90
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L58
