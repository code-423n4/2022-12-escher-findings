## Table of Contents
- Should Use Unchecked Block where Over/Underflow is not Possible
- Defined Variables Used Only Once
- X = X + Y costs less gass than X += Y
- Change Function Visibility Public to External
- Using Elements Smaller than 32 bytes (256 bits) Might Use More Gas
- Unnecessary Default Value Initialization

&ensp;
### Should Use Unchecked Block where Over/Underflow is not Possible

#### Issue
Since Solidity 0.8.0, all arithmetic operations revert on overflow and underflow by default.
However in places where overflow and underflow is not possible, it is better to use unchecked block to reduce the gas usage.
Reference: https://docs.soliditylang.org/en/v0.8.15/control-structures.html#checked-or-unchecked-arithmetic

#### PoC
1. Increments at OpenEdition.sol (line 66) can be unchecked 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66
Change to
```solidity
        for (uint24 x = temp.currentId + 1; x <= newId;) {
            nft.mint(msg.sender, x);
            unchecked { x++; }
        }
```

2. Increments at LPDA.sol (line 73) can be unchecked 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73
Change to
```solidity
        for (uint256 x = temp.currentId + 1; x <= newId;) {
            nft.mint(msg.sender, x);
            unchecked { x++; }
        }
```

3. Increments at FixedPrice.sol (line 65) can be unchecked 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65
Change to
```solidity
        for (uint48 x = sale_.currentId + 1; x <= newId;) {
            nft.mint(msg.sender, x);
            unchecked { x++; }
        }
```

#### Mitigation
Wrap the code with uncheck like described in above PoC. 

&ensp;
### Defined Variables Used Only Once

#### Issue
Certain variables is defined even though they are used only once.
Remove these unnecessary variables to save gas.
For cases where it will reduce the readability, one can use comments to help describe
what the code is doing.

#### PoC
1. Remove `timeElapsed` variable of getPrice() of LPDA.sol
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L123-L124
Mitigation:
Delete line 123 and replace line 124 like shown below
```solidity
return temp.startPrice - (temp.dropPerSecond * (end > block.timestamp ? block.timestamp - start : end - start));
```

2. Remove `nft` variable of buy() of LPDA.sol
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L61
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L74
Mitigation:
Delete line 61 and replace line 74 like shown below
```solidity
IEscher721(temp.edition).mint(msg.sender, x);
```

3. Remove `nft` variable of buy() of OpenEdition.sol
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L60
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L67
Mitigation:
Delete line 60 and replace line 67 like shown below
```solidity
IEscher721(temp.edition).mint(msg.sender, x);
```

4. Remove `nft` variable of buy() of FixedPrice.sol
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L59
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L66
Mitigation:
Delete line 59 and replace line 66 like shown below
```solidity
IEscher721(sale_.edition).mint(msg.sender, x);
```

#### Mitigation
Don't define variable that is used only once.
Details are listed on above PoC.

&ensp;
### X = X + Y costs less gass than X += Y

#### Issue
X = X + Y costs less gass than X += Y for state variables

#### PoC
Total of 3 instance found.
It saves 8 gas per instances
```solidity
./LPDA.sol:66:        amountSold += amount;
./LPDA.sol:70:        receipts[msg.sender].amount += amount;
./LPDA.sol:71:        receipts[msg.sender].balance += uint80(msg.value);
```

&ensp;
### Change Function Visibility Public to External

#### Issue
If the function is not called internally, it is cheaper to set your function visibility to external instead of public.

#### PoC
Total of 19 instances found.

1. LPDAFactory.sol:setFeeReceiver()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L46

2. FixedPriceFactory.sol:setFeeReceiver()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L42

3. OpenEditionFactory.sol:setFeeReceiver()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L42

4. Escher.sol:addCreator()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L27

5. Escher.sol:safeTransferFrom()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L38

6. Escher.sol:safeTransferFrom()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L49

7. OpenEdition.sol:finalize()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L89

8. OpenEdition.sol:timeLeft()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L107

9. OpenEdition.sol:available()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L116

10. LPDA.sol:available()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L138

11. FixedPrice.sol:available()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L101

12. LPDA.sol:refund()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L99

13. FixedPrice.sol:getPrice()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L86

14. OpenEdition.sol:getPrice()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L97

15. LPDA.sol:editionContract()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L133

16. FixedPrice.sol:editionContract()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L96

17. OpenEdition.sol:editionContract()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L112

18. LPDA.sol:lowestPrice()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L142

19. Escher721.sol:updateURIDelegate()
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L57

#### Mitigation
Change the function visibility to external.

&ensp;
### Using Elements Smaller than 32 bytes (256 bits) Might Use More Gas

#### Issue
Since EVM operates on 32 bytes at a time, if the element is smaller than that, the EVM must use more operations
in order to reduce the elements from 32 bytes to specified size. Therefore it is more gas saving to use 32 bytes 
unless it is used in a struct or state variable in order to reduce storage slot. 

Reference: https://docs.soliditylang.org/en/v0.8.15/internals/layout_in_storage.html

#### PoC
Total of 8 instances found.
```
./OpenEdition.sol:64:        uint24 newId = amount + temp.currentId;
./OpenEdition.sol:66:        for (uint24 x = temp.currentId + 1; x <= newId; x++) {
./LPDA.sol:36:    uint48 public amountSold = 0;
./LPDA.sol:67:        uint48 newId = amount + temp.currentId;
./LPDA.sol:102:        uint80 owed = r.balance - price;
./FixedPrice.sol:62:        uint48 newId = uint48(_amount) + sale_.currentId;
./FixedPrice.sol:65:        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
./Escher721.sol:66:        uint96 feeNumerator
```

#### Mitigation
I suggest using uint256 instead of anything smaller and downcast where needed.

&ensp;
### Unnecessary Default Value Initialization

#### Issue
When variable is not initialized, it will have its default values.
For example, 0 for uint, false for bool and address(0) for address.
Reference: https://docs.soliditylang.org/en/v0.8.15/control-structures.html#scoping-and-declarations

#### PoC
```solidity
LPDA.sol:36:    uint48 public amountSold = 0;
```

#### Mitigation
I suggest removing default value initialization.
```solidity
LPDA.sol:36:    uint48 public amountSold;
```

&ensp;