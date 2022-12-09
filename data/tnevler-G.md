# Report
## Gas Optimizations ##
### [G-1]: The increment in for loop post condition can be made unchecked
**Context:**

```
src\minters\FixedPrice.sol:
  65:         for (uint48 x = sale_.currentId + 1; x <= newId; x++) { 
```
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65

```
src\minters\LPDA.sol:
  73:         for (uint256 x = temp.currentId + 1; x <= newId; x++) { 
```
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73

```
src\minters\OpenEdition.sol:
  66:         for (uint24 x = temp.currentId + 1; x <= newId; x++) { 
```
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66

**Description:**

[This](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked) can save 30-40 gas per loop iteration.

**Recommendation:**

Example how to fix. Change:
```
for (uint256 i = 0; i < orders.length; ++i) {
   // Do the thing
}
```

To:
```
for (uint256 i = 0; i < orders.length;) {
   // Do the thing
   unchecked { ++i; }
}
```

### [G-2]: Place subtractions where the operands can't underflow in unchecked {} block  
**Context:**  
```
src\minters\LPDA.sol:
   86:          temp.saleReceiver.transfer(totalSale - fee); // unchecked {totalSale - fee}

  123:         uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start; // unchecked {block.timestamp - start}

  123:         uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start; // unchecked {end - start}
```
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L86  

**Description:**  
Some gas can be saved by using an unchecked {} block if an underflow isn't possible because of a previous require() or if-statement.

### [G-3]: Not re-read state variable from the storage
**Context:** 
``` 
src\minters\OpenEdition.sol:
  63:         require(amount * sale.price == msg.value, "WRONG PRICE");
```

**Description:**
Every reading from storage costs about 100 gas while every reading from memory costs only 3 gas. State variable sale is already cached in local [memory](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L59).

**Recommendation:**  
Change to:  
require(amount * temp.price == msg.value, "WRONG PRICE");


### [G-4]: Storage pointer to a structure is cheaper than copying each value of the structure into memory, same for array and mapping
**Context:** 
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L58
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L59
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L90
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L60
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L100
1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L118

**Description:**
Every time you copy a storage struct/array/mapping to a memory variable, you are literally copying each member by reading it from storage, which is expensive. And when you use the storage keyword, you are just storing a pointer to the storage, which is much cheaper.

**Recommendation:**  
Change ```memory``` to ```storage```:
```
Sale storage sale_ = sale;
```
