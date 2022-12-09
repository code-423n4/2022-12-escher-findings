|  | Issue | Instances | Saved Gas | Overall gas change from forge snapshot --diff |
| --- | --- | --- | --- | --- |
| [G-01] | Refactor Sale struct to avoid using unnecessary storage slot | 3 | 73328 | 972780 |
| [G-02] | Cache repeated parameters from Sale in buy() function | 2 | 409 | 10066 |
| [G-03] | Use cached value from memory rather than read storage | 1 | 115 | 1380 |
| [G-04] | Removing cache Sale to memory saves gas | 1 | 254 | 6775 |
| [G-05] | Increment in the for loop’s postcondition can be made unchecked{++i}/unchecked{i++} | 3 | 215 | 5855 |
| [G-06] | Simplify / Refactor _end function | 2 | 325 | 5460 |
| [G-07] | Change for loop behavior by removing add (+1) and ++x is more gas efficient  | 3 | 304 | 4025 |
| [G-08] | <x> += <y> cost more gas than <x> = <x> + <y> | 1 | 42 | 756 |
|  |  |  |  |  |

# Gas Report

Total gas saved: **74992**

Overall gas change from forge snapshot: **1002920**

```diff
test_RevertsWhenEnded_Buy() (gas: -26155 (-3.807%)) 
test_SellsOut_Buy() (gas: -26157 (-3.824%)) 
test_LPDA() (gas: -26988 (-3.850%)) 
test_RevertsWhenSoldOut_Buy() (gas: -27100 (-3.887%)) 
test_RevertsWhenMintedOut_Buy() (gas: -26564 (-4.306%)) 
test_WhenMintsOut_Buy() (gas: -26256 (-4.318%)) 
test_Refund() (gas: -25377 (-6.436%)) 
test_WhenNotOver_Refund() (gas: -25377 (-6.436%)) 
test_RevertsWhenAlreadyRefunded_Refund() (gas: -25731 (-6.472%)) 
test_RevertsWhenTooSoon_Buy() (gas: -25463 (-6.477%)) 
test_RevertsWhenTooLate_Cancel() (gas: -25002 (-6.481%)) 
test_RevertsWhenNotOwner_Cancel() (gas: -25007 (-6.483%)) 
test_RevertsWhenNoRefund_Refund() (gas: -25377 (-6.507%)) 
test_Buy() (gas: -25007 (-6.560%)) 
test_RevertsWhenTooLittleValue_Buy() (gas: -25818 (-6.667%)) 
test_WhenEnded_Finalize() (gas: -24907 (-7.069%)) 
test_RevertsWhenNotAdmin_CreateSale() (gas: -2025 (-7.794%)) 
test_RevertsWhenTooSoon_Buy() (gas: -24797 (-7.880%)) 
test_RevertsWhenEnded_Buy() (gas: -24813 (-7.886%)) 
test_RevertsTooMuchValue_Buy() (gas: -24924 (-7.926%)) 
test_RevertsTooMuchValue_Buy() (gas: -25137 (-8.003%)) 
test_RevertsWhenTooMany_Buy() (gas: -25155 (-8.006%)) 
test_RevertsWhenNotOwner_TransferOwnership() (gas: -24708 (-8.010%)) 
test_RevertsWhenTooSoon_Buy() (gas: -25239 (-8.025%)) 
test_RevertsWhenNotOwner_TransferOwnership() (gas: -24758 (-8.031%)) 
test_RevertsWhenNotOwner_Cancel() (gas: -24708 (-8.035%)) 
test_RevertsWhenTooLate_Cancel() (gas: -24720 (-8.040%)) 
test_RevertsWhenTooLate_Cancel() (gas: -24864 (-8.091%)) 
test_RevertsWhenTooLittleValue_Buy() (gas: -24924 (-8.096%)) 
test_RevertsWhenNotOwner_Cancel() (gas: -24869 (-8.104%)) 
test_Buy() (gas: -24708 (-8.156%)) 
test_RevertsWhenNotEnded_Finalize() (gas: -25144 (-8.162%)) 
test_RevertsWhenTooLittleValue_Buy() (gas: -25137 (-8.176%)) 
test_Buy() (gas: -24847 (-8.207%)) 
test_RevertsWhenNotAdmin_CreateSale() (gas: -2078 (-8.743%)) 
test_RevertsWhenNotAdmin_CreateSale() (gas: -2093 (-8.814%)) 
test_Cancel() (gas: -24400 (-9.382%)) 
test_Cancel() (gas: -24208 (-9.592%)) 
test_Cancel() (gas: -24699 (-10.928%)) 
test_RevertsWhenInitialized_Initialize() (gas: -1971 (-10.952%)) 
test_CreateSale() (gas: -24216 (-11.069%)) 
test_RevertsWhenInitialized_Initialize() (gas: -2137 (-11.845%)) 
test_CreateSale() (gas: -24277 (-12.900%)) 
test_CreateSale() (gas: -24316 (-12.913%)) 
test_RevertsWhenInitialized_Buy() (gas: -2089 (-13.919%)) 
test_RevertsWhenInitialized_Buy() (gas: -4392 (-29.262%)) 
test_RevertsWhenInitialized_Finalize() (gas: -4414 (-29.460%)) 
Overall gas change: -1002920 (-413.087%)
```

### [G-01] Refactor Sale struct to avoid using unnecessary storage slot

We can use `uint32` for time variables instead of `uint96` since it will be enough until the year **2106** but can save a decent amount of gas.

```diff
file: src/minters/LPDA.sol

struct Sale {
		// slot 1
		uint48 currentId;
		uint48 finalId;
		address edition;
		// slot 2
		uint80 startPrice;
		uint80 finalPrice;
+   uint32 startTime;
+   uint32 endTime;
-		uint80 dropPerSecond;
		// slot 3
-		uint96 endTime;
+   uint80 dropPerSecond;
		address payable saleReceiver;
-		// slot 4
-		uint96 startTime;
}
```

`test_Buy() (gas: -24451 (-6.415%))`

`Overall gas change: -370656 (-99.469%)`

Same here, we can use `uint32` for the time variables

```diff
file: src/minters/OpenEdition.sol

struct Sale {
	  // slot 1
	  uint72 price;
	  uint24 currentId;
	  address edition;
	  // slot 2
-	  uint96 startTime;
+	  uint32 startTime;
-   uint96 endTime;
+   uint32 endTime;
	  address payable saleReceiver;
-	  // slot 3
-	  uint96 endTime;
}
```

`test_Buy() (gas: -24420 (-8.061%))`

`Overall gas change: -301789 (-149.642%)`

In FixedPrice.sol we use `uint32` for the `startTime` variable and `uint32` for `currentId` and `finalId` . Since in OpenEdition.sol use `uint24` for `currentId`

```diff
file: src/minters/FixedPrice.sol

struct Sale {
	  // slot 1
-	  uint48 currentId;
-	  uint48 finalId;
+   uint32 currentId;
+	  uint32 finalId;
+   uint32 startTime;
	  address edition;
	  // slot 2
	  uint96 price;
	  address payable saleReceiver;
-	  // slot 3
-	  uint96 startTime;
}
```

`test_Buy() (gas: -24457 (-8.078%))`

`Overall gas change: -300344 (-126.975%)`

### [G-02] Cache repeated parameters from Sale in `buy()` function.

Instead of caching the entire Sale object, only cache the `currentId` and `finalId` parameters in the function. This will save on gas consumption and improve efficiency. 

```diff
file: src/minters/FixedPrice.sol

function buy(uint256 _amount) external payable {
-        Sale memory sale_ = sale;
+        uint48 currentId = sale.currentId;
+        uint48 finalId = sale.finalId;
-        IEscher721 nft = IEscher721(sale.edition);
+        IEscher721 nft = IEscher721(sale.edition);
-        require(block.timestamp >= sale_.startTime, "TOO SOON");
+        require(block.timestamp >= sale.startTime, "TOO SOON");
-        require(_amount * sale_.price == msg.value, "WRONG PRICE");
+        require(_amount * sale.price == msg.value, "WRONG PRICE");
-        uint48 newId = uint48(_amount) + sale_.currentId;
+        uint48 newId = uint48(_amount) + currentId;
-        require(newId <= sale_.finalId, "TOO MANY");
+        require(newId <= finalId, "TOO MANY");

-        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
+        for (uint48 x = currentId + 1; x <= newId; x++) {
             nft.mint(msg.sender, x);
         }

         sale.currentId = newId;

         emit Buy(msg.sender, _amount, msg.value, sale);

-        if (newId == sale_.finalId) _end(sale);
+        if (newId == finalId) _end(sale);
    }

```

`test_Buy() (gas: -189 (-0.062%))`

```diff
file: src/minters/LPDA.sol

function buy(uint256 _amount) external payable {
    uint48 amount = uint48(_amount);
-    Sale memory temp = sale;
+    uint48 currentId = sale.currentId;
+    uint48 finalId = sale.finalId;
-    IEscher721 nft = IEscher721(temp.edition);
+    IEscher721 nft = IEscher721(sale.edition);
-    require(block.timestamp >= temp.startTime, "TOO SOON");
+    require(block.timestamp >= sale.startTime, "TOO SOON");
     uint256 price = getPrice();
     require(msg.value >= amount * price, "WRONG PRICE");

     amountSold += amount;
-    uint48 newId = amount + temp.currentId;
+    uint48 newId = amount + currentId;
-    require(newId <= temp.finalId, "TOO MANY");
+    require(newId <= finalId, "TOO MANY");

     receipts[msg.sender].amount += amount;
     receipts[msg.sender].balance += uint80(msg.value);

-    for (uint256 x = temp.currentId + 1; x <= newId; x++) {
+    for (uint256 x = currentId + 1; x <= newId; x++) {
         nft.mint(msg.sender, x);
     }

     sale.currentId = newId;

-    emit Buy(msg.sender, amount, msg.value, temp);
+    emit Buy(msg.sender, amount, msg.value, sale);
-    if (newId == temp.finalId) {
+    if (newId == finalId) {
         sale.finalPrice = uint80(price);
         uint256 totalSale = price * amountSold;
         uint256 fee = totalSale / 20;
         ISaleFactory(factory).feeReceiver().transfer(fee);
-        temp.saleReceiver.transfer(totalSale - fee);
+        sale.saleReceiver.transfer(totalSale - fee);
         _end();
     }
}
```

`test_Buy() (gas: -220 (-0.058%))` and `test_LPDA() (gas: -319 (-0.046%))`

### [G-03] Use cached value from memory rather than read storage

```diff
file: src/minters/OpenEdition.sol

function buy(uint256 _amount) external payable {
		uint24 amount = uint24(_amount);
		Sale memory temp = sale;
		IEscher721 nft = IEscher721(temp.edition);
		require(block.timestamp >= temp.startTime, "TOO SOON");
		require(block.timestamp < temp.endTime, "TOO LATE");
-		require(amount * sale.price == msg.value, "WRONG PRICE");
+		require(amount * temp.price == msg.value, "WRONG PRICE");
		uint24 newId = amount + temp.currentId;
		
		for (uint24 x = temp.currentId + 1; x <= newId; x++) {
		    nft.mint(msg.sender, x);
		}
		sale.currentId = newId;
		
		emit Buy(msg.sender, amount, msg.value, temp);
}
```

`test_Buy() (gas: -115 (-0.038%))`

### [G-04] Removing cache Sale to memory saves gas

it will save gas in `buy()` and `refund()` functions

```diff
file: src/minters/LPDA.sol

function getPrice() public view returns (uint256) {
-		Sale memory temp = sale;
-		(uint256 start, uint256 end) = (temp.startTime, temp.endTime);
+		(uint256 start, uint256 end) = (sale.startTime, sale.endTime);
		if (block.timestamp < start) return type(uint256).max;
-		if (temp.currentId == temp.finalId) return temp.finalPrice;
+		if (sale.currentId == sale.finalId) return temp.finalPrice;
		
-		uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;
-		uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;
-		return temp.startPrice - (temp.dropPerSecond * timeElapsed);
+   return sale.startPrice - (sale.dropPerSecond * timeElapsed);
}
```

`test_Buy() (gas: -254 (-0.067%))` and `test_Refund() (gas: -507 (-0.129%))`

### [G-05] I****ncrement in the for loop’s post condition can be made unchecked****

The `newId`variable is calculated using addition, which means it cannot overflow. (In Solidity version 0.8.0 and higher)

```diff
file: src/minters/FixedPrice.sol

function buy(uint256 _amount) external payable {
		...

-    for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
+    for (uint48 x = sale_.currentId + 1; x <= newId; ) {
        nft.mint(msg.sender, x);
+       unchecked {
+            x++;
+       }
    }
		...
}
```

`test_Buy() (gas: -78 (-0.026%))`

```diff
file: src/minters/LPDA.sol

function buy(uint256 _amount) external payable {
		...

-    for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
+    for (uint48 x = sale_.currentId + 1; x <= newId; ) {
        nft.mint(msg.sender, x);
+       unchecked {
+            x++;
+       }
    }
		...
}
```

`test_Buy() (gas: -59 (-0.015%))` and `test_LPDA() (gas: -590 (-0.084%))`

```diff
file: src/minters/OpenEdition.sol

function buy(uint256 _amount) external payable {
		...

-    for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
+    for (uint48 x = sale_.currentId + 1; x <= newId; ) {
        nft.mint(msg.sender, x);
+       unchecked {
+            x++;
+       }
    }
		...
}
```

`test_Buy() (gas: -78 (-0.026%))`

### [G-06] Simplify `_end` function

Also in OpenEdition.sol we can remove and unnecessary struct cache in `finalize()` function.

```diff
file: src/minters/OpenEdition.sol

function cancel() external onlyOwner {
    require(block.timestamp < sale.startTime, "TOO LATE");
-   _end(sale);
+   _end();
}

function finalize() public {
-   Sale memory temp = sale;
-   require(block.timestamp >= temp.endTime, "TOO SOON");
+   require(block.timestamp >= sale.endTime, "TOO SOON");
    ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
-   _end(temp);
+   _end();
}

- function _end(Sale memory _sale) internal {
+ function _end() internal {
-	    emit End(_sale);
+	    emit End(sale);
-	    selfdestruct(_sale.saleReceiver);
+	    selfdestruct(sale.saleReceiver);
	}
```

`test_Cancel() (gas: -304 (-0.135%))`

In FixedPrice.sol refactor `_end` function save 21 gas.

```diff
file: src/minters/FixedPrice.sol

function cancel() external onlyOwner {
    require(block.timestamp < sale.startTime, "TOO LATE");
-   _end(sale);
+   _end();
}

function buy(uint256 _amount) external payable {
...
-		if (newId == sale_.finalId) _end(sale);
+		if (newId == sale_.finalId) _end();
}

- function _end(Sale memory _sale) internal {
+ function _end() internal {
+     Sale memory temp = sale;
-	    emit End(_sale);
+     emit End(temp);
	    ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
-	    selfdestruct(_sale.saleReceiver);
+    selfdestruct(temp.saleReceiver);
	}
```

`test_Cancel() (gas: -21 (-0.008%))`

### [G-07] Change for loop behavior by removing add (+1) and ++x is more gas efficient

```diff
file: src/minters/FixedPrice.sol

function buy(uint256 _amount) external payable {
				...

-       for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
+       for (uint48 x = sale_.currentId; x < newId; ++x) {
            nft.mint(msg.sender, x);
        }
				...
    }
```

`test_Buy() (gas: -105 (-0.035%))`

```diff
file: src/minters/LPDA.sol

function buy(uint256 _amount) external payable {
				...

-       for (uint256 x = temp.currentId + 1; x <= newId; x++) {
+       for (uint256 x = temp.currentId; x < newId; ++x) {
            nft.mint(msg.sender, x);
        }

				...
    }
```

`test_Buy() (gas: -97 (-0.025%))` and `test_LPDA() (gas: -210 (-0.030%))`

```diff
file: src/minters/OpenEdition.sol

function buy(uint256 _amount) external payable {
		...

-   for (uint24 x = temp.currentId + 1; x <= newId; x++) {
+   for (uint24 x = temp.currentId; x < newId; ++x) {
        nft.mint(msg.sender, x);
    }

		...
}
```

`test_Buy() (gas: -102 (-0.034%))`

### [G-08] `<x> += <y>` cost more gas than `<x> = <x> + <y>`

```diff
file: src/minters/LPDA.sol

function buy(uint256 _amount) external payable {
		...
		require(msg.value >= amount * price, "WRONG PRICE");

-		amountSold += amount;
+		amountSold = amountSold + amount;
		uint48 newId = amount + temp.currentId;

		...
}
```

`test_Buy() (gas: -42 (-0.011%))`