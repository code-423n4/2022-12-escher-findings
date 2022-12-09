## FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

### Description:

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for 
legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3)
,DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

Affected instances:
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L57
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L51
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L64
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L72
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L50
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L21
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L27
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L75
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L92


## USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS

### Description:

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, 
which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional 
MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. 
The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, 
is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

Affected instances:

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L58
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L59
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L90
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L100
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L118
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L60


##  ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

### Description:

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

Affected instances:
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73

## <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES

### Description:
Using the addition operator instead of plus-equals saves 113 gas

Affected instances:
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L66
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L70
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L71



