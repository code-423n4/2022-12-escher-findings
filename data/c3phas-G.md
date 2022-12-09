## FINDINGS
NB: *Some functions have been truncated where neccessary to just show affected parts of the code*

### Declare a constant variable  for the type max rather than evaluate it everytime
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L46
```solidity
File:/src/minters/FixedPrice.sol
46:        sale = Sale(0, 0, address(0), type(uint96).max, payable(0), type(uint96).max);
```

```diff
diff --git a/src/minters/FixedPrice.sol b/src/minters/FixedPrice.sol
index 4e2cb9e..bee73c5 100644
--- a/src/minters/FixedPrice.sol
+++ b/src/minters/FixedPrice.sol
@@ -26,6 +26,8 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
     /// @notice sale info for this fixed price edition
     Sale public sale;

+    uint96 private constant UINT96_MAX = type(uint96).max;
+
     /// @notice Event emitted when sale created
     /// @param _saleInfo the sale info for the edition
     event Start(Sale _saleInfo);
@@ -43,7 +45,7 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
     constructor() {
         factory = msg.sender;
         _disableInitializers();
-        sale = Sale(0, 0, address(0), type(uint96).max, payable(0), type(uint96).max);
+        sale = Sale(0, 0, address(0), UINT96_MAX, payable(0), UINT96_MAX);
     }

```


https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L42-L53
```solidity
File: /src/minters/OpenEdition.sol
42:    constructor() {

48:            address(0),
49:            type(uint96).max,
50:            payable(0),
51:            type(uint96).max
52:        );
53:    }
```

### Cache storage values in memory to minimize SLOADs
The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

NB: *Some functions have been truncated where neccessary to just show affected parts of the code*

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L27-L30
###  Generative.sol.tokenToSeed(): seedBase should be cached(saves 1 SLOAD ~100 gas)
```solidity
File: /src/uris/Generative.sol
27:    function tokenToSeed(uint256 _tokenId) internal view returns (bytes32) {
28:        require(seedBase != bytes32(0), "SEED BASE NOT SET");
29:        return keccak256(abi.encodePacked(_tokenId, seedBase));
30:    }
```

```diff
diff --git a/src/uris/Generative.sol b/src/uris/Generative.sol
index db09737..2d24c12 100644
--- a/src/uris/Generative.sol
+++ b/src/uris/Generative.sol
@@ -25,7 +25,8 @@ contract Generative is Unique {
     }

     function tokenToSeed(uint256 _tokenId) internal view returns (bytes32) {
-        require(seedBase != bytes32(0), "SEED BASE NOT SET");
-        return keccak256(abi.encodePacked(_tokenId, seedBase));
+        bytes32 _seedBase = seedBase;
+        require(_seedBase != bytes32(0), "SEED BASE NOT SET");
+        return keccak256(abi.encodePacked(_tokenId, _seedBase));
     }
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L50-L53
### FixedPrice.sol.cancel(): sale should be cached
```solidity
File: /src/minters/FixedPrice.sol
50:    function cancel() external onlyOwner {
51:        require(block.timestamp < sale.startTime, "TOO LATE");
52:        _end(sale);
53:    }
```

```diff
diff --git a/src/minters/FixedPrice.sol b/src/minters/FixedPrice.sol
index 4e2cb9e..e8819b2 100644
--- a/src/minters/FixedPrice.sol
+++ b/src/minters/FixedPrice.sol
@@ -48,8 +48,9 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {

     /// @notice Owner can cancel current sale
     function cancel() external onlyOwner {
-        require(block.timestamp < sale.startTime, "TOO LATE");
-        _end(sale);
+        Sale  _sale = sale;
+        require(block.timestamp < _sale.startTime, "TOO LATE");
+        _end(_sale);
     }
```


https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L75-L78

```solidity
File:/src/minters/OpenEdition.sol
75:    function cancel() external onlyOwner {
76:        require(block.timestamp < sale.startTime, "TOO LATE");
77:        _end(sale);
78:    }
```

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..bd90e0f 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -73,8 +73,9 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {

     /// @notice cancel a fixed price sale
     function cancel() external onlyOwner {
-        require(block.timestamp < sale.startTime, "TOO LATE");
-        _end(sale);
+        Sale storage _sale = sale;
+        require(block.timestamp < _sale.startTime, "TOO LATE");
+        _end(_sale);
     }

```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L107-L109
```solidity
File: /src/minters/OpenEdition.sol
107:    function timeLeft() public view returns (uint256) {
108:        return sale.endTime > block.timestamp ? sale.endTime - block.timestamp : 0;
109:    }
```

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..fe819c0 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -105,7 +105,8 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {

     /// @notice how much time is left in the sale
     function timeLeft() public view returns (uint256) {
-        return sale.endTime > block.timestamp ? sale.endTime - block.timestamp : 0;
+        uint96 saleEndTime = sale.endTime;
+        return saleEndTime > block.timestamp ? saleEndTime - block.timestamp : 0;
     }
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L58-L89
```solidity
File: /src/minters/LPDA.sol
58:    function buy(uint256 _amount) external payable {
    
66:        amountSold += amount;

81:        if (newId == temp.finalId) {
82:            sale.finalPrice = uint80(price);
83:            uint256 totalSale = price * amountSold;

88:        }
89:    }
```

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..6cf6952 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -62,8 +62,8 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
         require(block.timestamp >= temp.startTime, "TOO SOON");
         uint256 price = getPrice();
         require(msg.value >= amount * price, "WRONG PRICE");
-
-        amountSold += amount;
+        uint48 _amountSold = amountSold;
+        amountSold = _amountSold + amount;
         uint48 newId = amount + temp.currentId;
         require(newId <= temp.finalId, "TOO MANY");

@@ -80,7 +80,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {

         if (newId == temp.finalId) {
             sale.finalPrice = uint80(price);
-            uint256 totalSale = price * amountSold;
+            uint256 totalSale = price * _amountSold;
             uint256 fee = totalSale / 20;
             ISaleFactory(factory).feeReceiver().transfer(fee);
             temp.saleReceiver.transfer(totalSale - fee);
```

### Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time.
**Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. 
As an example, instead of repeatedly calling ```someMap[someIndex]```, save its reference like this: ```SomeStruct storage someStruct = someMap[someIndex]``` and use it.

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L70-L71
```solidity
File: /src/minters/LPDA.sol
        receipts[msg.sender].amount += amount;
        receipts[msg.sender].balance += uint80(msg.value);
```

### Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L57-L74
```solidity
File: /src/minters/FixedPrice.sol
57:    function buy(uint256 _amount) external payable {
58:        Sale memory sale_ = sale;
```

```diff
diff --git a/src/minters/FixedPrice.sol b/src/minters/FixedPrice.sol
index 4e2cb9e..363258b 100644
--- a/src/minters/FixedPrice.sol
+++ b/src/minters/FixedPrice.sol
@@ -55,7 +55,7 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
     /// @notice buy from a fixed price sale after the sale starts
     /// @param _amount the amount of editions to buy
     function buy(uint256 _amount) external payable {
-        Sale memory sale_ = sale;
+        Sale storage sale_ = sale;
         IEscher721 nft = IEscher721(sale_.edition);
         require(block.timestamp >= sale_.startTime, "TOO SOON");
         require(_amount * sale_.price == msg.value, "WRONG PRICE");
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L57-L72
```solidity
File:/src/minters/OpenEdition.sol
57:    function buy(uint256 _amount) external payable {
58:        uint24 amount = uint24(_amount);
59:        Sale memory temp = sale;
```

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..7133557 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -56,7 +56,7 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
     /// @param _amount the amount of editions to buy
     function buy(uint256 _amount) external payable {
         uint24 amount = uint24(_amount);
-        Sale memory temp = sale;
+        Sale storage temp = sale;
         IEscher721 nft = IEscher721(temp.edition);
         require(block.timestamp >= temp.startTime, "TOO SOON");
         require(block.timestamp < temp.endTime, "TOO LATE");
```


https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L89-L94
### Saves 1587 gas on average

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 864    | 17848   | 6864 | 45816 |
| After  | 418    | 16261   | 2418 | 45949 |


```solidity
File: /src/minters/OpenEdition.sol
89:    function finalize() public {
90:        Sale memory temp = sale;
91:        require(block.timestamp >= temp.endTime, "TOO SOON");
92:        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
93:        _end(temp);
94:    }
```

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..04d1464 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -87,7 +87,7 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {

     /// @notice finish an open edition and payout the artist
     function finalize() public {
-        Sale memory temp = sale;
+        Sale storage temp = sale;
         require(block.timestamp >= temp.endTime, "TOO SOON");
         ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
         _end(temp);
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L58-L89

```solidity
File: /src/minters/LPDA.sol
58:    function buy(uint256 _amount) external payable {
59:        uint48 amount = uint48(_amount);
60:        Sale memory temp = sale;
```

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..bcf2b94 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -57,7 +57,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
     /// @param _amount the amount of editions to buy
     function buy(uint256 _amount) external payable {
         uint48 amount = uint48(_amount);
-        Sale memory temp = sale;
+        Sale storage temp = sale;
         IEscher721 nft = IEscher721(temp.edition);
         require(block.timestamp >= temp.startTime, "TOO SOON");
         uint256 price = getPrice();
         
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L99-L106
### Save 26 gas on average

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 2148    | 7221   | 9174 | 9342 |
| After  | 2176    | 7195   | 9121 | 9289 |

```solidity
File: /src/minters/LPDA.sol
99:    function refund() public {
100:        Receipt memory r = receipts[msg.sender];
```

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..5cc7bde 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -97,11 +97,11 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {

     /// @notice allow a buyer to get a refund on the current price difference
     function refund() public {
-        Receipt memory r = receipts[msg.sender];
+        Receipt storage r = receipts[msg.sender];
         uint80 price = uint80(getPrice()) * r.amount;
         uint80 owed = r.balance - price;
         require(owed > 0, "NOTHING TO REFUND");
-        receipts[msg.sender].balance = price;
+        r.balance = price;
         payable(msg.sender).transfer(owed);
     }
```


https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L117-L125

### Save 233 gas

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 1501    | 1501   | 1501 | 1501 |
| After  | 1268    | 1268   | 1268 | 1268 |

```solidity
File: /src/minters/LPDA.sol
117:    function getPrice() public view returns (uint256) {
118:        Sale memory temp = sale;
```

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..65a1918 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -115,7 +115,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {

     /// @notice the price of the sale
     function getPrice() public view returns (uint256) {
-        Sale memory temp = sale;
+        Sale storage temp = sale;
```

### x += y costs more gas than x = x + y for state variables

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L66
```solidity
File: /src/minters/LPDA.sol
66:        amountSold += amount;
```

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..5de2d13 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -63,7 +63,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
         uint256 price = getPrice();
         require(msg.value >= amount * price, "WRONG PRICE");

-        amountSold += amount;
+        amountSold = amountSold + amount;
         uint48 newId = amount + temp.currentId;
         require(newId <= temp.finalId, "TOO MANY");
```

### Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

   When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L64-L69

```solidity
File: /src/Escher721.sol

//@audit: uint96 feeNumerator
64:    function setDefaultRoyalty(
65:        address receiver,
66:        uint96 feeNumerator
67:    ) public onlyRole(DEFAULT_ADMIN_ROLE) {
68:        _setDefaultRoyalty(receiver, feeNumerator);
69:    }
```
### Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L86
```solidity
File: /src/minters/LPDA.sol
86:            temp.saleReceiver.transfer(totalSale - fee);
```


```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..f1e4277 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -83,7 +83,10 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
             uint256 totalSale = price * amountSold;
             uint256 fee = totalSale / 20;
             ISaleFactory(factory).feeReceiver().transfer(fee);
-            temp.saleReceiver.transfer(totalSale - fee);
+            unchecked {
+               temp.saleReceiver.transfer(totalSale - fee);
+            }
             _end();
         }
     }
```

`totalSale - fee` would never underflow since `fee` is equivalent to `totalSale / 20` see [Line 84](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L84)

[see resource](https://github.com/ethereum/solidity/issues/10695)


### Using unchecked blocks to save gas - Increments in for loop can be unchecked  ( save 30-40 gas per loop iteration)
The majority of Solidity for loops increment a uint256 variable that starts at 0. These increment operations never need to be checked for over/underflow because the variable will never reach the max number of uint256 (will run out of gas long before that happens). The default over/underflow check wastes gas in every iteration of virtually every for loop . eg.

e.g Let's work with a sample loop below.

```solidity
for(uint256 i; i < 10; i++){
//doSomething
}

```
can be written as shown below.
```solidity
for(uint256 i; i < 10;) {
  // loop logic
  unchecked { i++; }
}
```

We can also write  it as an inlined function like below.

```solidity
function inc(i) internal pure returns (uint256) {
  unchecked { return i + 1; }
}
for(uint256 i; i < 10; i = inc(i)) {
  // doSomething
}
```

**Affected code**
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65-L67
```solidity
File: /src/minters/FixedPrice.sol
65:        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
66:            nft.mint(msg.sender, x);
67:        }
```

```diff
diff --git a/src/minters/FixedPrice.sol b/src/minters/FixedPrice.sol
index 4e2cb9e..be2a3d7 100644
--- a/src/minters/FixedPrice.sol
+++ b/src/minters/FixedPrice.sol
@@ -62,8 +62,11 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
         uint48 newId = uint48(_amount) + sale_.currentId;
         require(newId <= sale_.finalId, "TOO MANY");

-        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
+        for (uint48 x = sale_.currentId + 1; x <= newId;) {
             nft.mint(msg.sender, x);
+            unchecked {
+                ++x;
+            }
         }
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66-L68
```solidity
File:/src/minters/OpenEdition.sol
66:        for (uint24 x = temp.currentId + 1; x <= newId; x++) {
67:            nft.mint(msg.sender, x);
68:        }
```

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..252fd65 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -63,8 +63,11 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
         require(amount * sale.price == msg.value, "WRONG PRICE");
         uint24 newId = amount + temp.currentId;

-        for (uint24 x = temp.currentId + 1; x <= newId; x++) {
+        for (uint24 x = temp.currentId + 1; x <= newId;) {
             nft.mint(msg.sender, x);
+            unchecked {
+                ++x;
+            }
         }
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73-L75
```solidity
File: /src/minters/LPDA.sol
73:        for (uint256 x = temp.currentId + 1; x <= newId; x++) {
74:            nft.mint(msg.sender, x);
75:        }
```

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..8bc1680 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -70,8 +70,11 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
         receipts[msg.sender].amount += amount;
         receipts[msg.sender].balance += uint80(msg.value);

-        for (uint256 x = temp.currentId + 1; x <= newId; x++) {
+        for (uint256 x = temp.currentId + 1; x <= newId;) {
             nft.mint(msg.sender, x);
+            unchecked {
+                ++x;
+            }
```

[see resource](https://github.com/ethereum/solidity/issues/10695)

