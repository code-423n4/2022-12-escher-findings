# Index
1. Optimize LPDA.getPrice()
2. I = I + (-) X is cheaper in gas cost than I += (-=) X
3. Optimize LPDA.buy
4. Unchecked operations in getPrice() when it's not possible to overflow/underflow
5. Use unchecked in for/while loops when it's not possible to overflow

# Details
## 1. Optimize LPDA.getPrice()
### Description
It's possible to save gas if move the second if to the beginning. There are situations where 
that if is true and it's not necessary to assig value to start & end local variables or evaluate the first if (block.timestamp < start).

```diff
    function getPrice() public view returns (uint256) {
        Sale memory temp = sale;
+       if (temp.currentId == temp.finalId) return temp.finalPrice;
        (uint256 start, uint256 end) = (temp.startTime, temp.endTime);
        if (block.timestamp < start) return type(uint256).max;
-       if (temp.currentId == temp.finalId) return temp.finalPrice;

        uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;
        return temp.startPrice - (temp.dropPerSecond * timeElapsed);
    }
```

### Lines in the code
[LPDA.sol#L117-L125](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L117-L125)


## 2. I = I + (-) X is cheaper in gas cost than I += (-=) X
### Description
When we are working with state variables is cheaper to use I = I + X than I += X.

### PoC
Original Code
Running 14 tests for test/LPDA.t.sol:LPDATest
[PASS] test_Buy() (gas: 381183)

Code after apply changes
Running 14 tests for test/LPDA.t.sol:LPDATest
[PASS] test_Buy() (gas: 381141)

There is 42 gas saved with this change for each instance. Due to there are 3 instances **the gas saved is 126**


### Lines in the code
[LPDA.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L66)
[LPDA.sol#L70](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L70)
[LPDA.sol#L71](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L71)


## 3. Optimize LPDA.buy
### Description
The declaration of the variable is made at the beginning of the function while its use is made later on. 
Between the declaration of the variable and its use, there are several requirements that may result in the variable never being used. 
By moving the declaration after the requires and before it is used, gas can be saved in some circunstances.

```diff
    function buy(uint256 _amount) external payable {
        uint48 amount = uint48(_amount);
        Sale memory temp = sale;
-       IEscher721 nft = IEscher721(temp.edition);
        require(block.timestamp >= temp.startTime, "TOO SOON");
        uint256 price = getPrice();
        require(msg.value >= amount * price, "WRONG PRICE");

        amountSold += amount;
        uint48 newId = amount + temp.currentId;
        require(newId <= temp.finalId, "TOO MANY");

        receipts[msg.sender].amount += amount;
        receipts[msg.sender].balance += uint80(msg.value);

+       IEscher721 nft = IEscher721(temp.edition);
        for (uint256 x = temp.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }
```
[LPDA.sol#L58-L75](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L58-L75)
[FixedPrice.sol#L59-L67](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L59-L67)


## 4. Unchecked operations in getPrice() when it's not possible to overflow/underflow
### Description
There are code where it's not possible to overflow/undeflow due to a IF or Require statement. In that case, it's possible
to save gas adding the unckecked block.

### Change proposed

```diff
    function getPrice() public view returns (uint256) {
        Sale memory temp = sale;
        (uint256 start, uint256 end) = (temp.startTime, temp.endTime);
        if (block.timestamp < start) return type(uint256).max;
        if (temp.currentId == temp.finalId) return temp.finalPrice;

-		uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;
+       uint256 timeElapsed;
+       unchecked{ timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;}
        return temp.startPrice - (temp.dropPerSecond * timeElapsed);
    }
```

### PoC

Original Code
| src/minters/LPDA.sol:LPDA contract |                 |        |        |        |         |
|------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                    | Deployment Size |        |        |        |         |
| 1252237                            | 6103            |        |        |        |         |
| Function Name                      | min             | avg    | median | max    | # calls |
| buy                                | 1149            | 125350 | 101393 | 295596 | 20      |
| cancel                             | 475             | 1912   | 627    | 4635   | 3       |
| getPrice                           | 1501            | 1501   | 1501   | 1501   | 1       |
| initialize                         | 143516          | 143516 | 143516 | 143516 | 15      |
| refund                             | 2148            | 7221   | 9174   | 9342   | 6       |

Code after apply changes
| src/minters/LPDA.sol:LPDA contract |                 |        |        |        |         |
|------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                    | Deployment Size |        |        |        |         |
| 1249637                            | 6090            |        |        |        |         |
| Function Name                      | min             | avg    | median | max    | # calls |
| buy                                | 1149            | 125291 | 101328 | 295530 | 20      |
| cancel                             | 475             | 1912   | 627    | 4635   | 3       |
| getPrice                           | 1435            | 1435   | 1435   | 1435   | 1       |
| initialize                         | 143516          | 143516 | 143516 | 143516 | 15      |
| refund                             | 2082            | 7166   | 9142   | 9276   | 6       |

There is **66 of gas saved** directly in getPrice due to the change. 
Due to getPrice is a use in other functions (buy and refund) there is a major gas saved because of each time
this functions use getPrice 66 of gas is saved.

### Lines in the code
[LPDA.sol#L117-L125](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L117-L125)


## 5. Use unchecked in for/while loops when it's not possible to overflow
### Description
Use unchecked { i++; } or unchecked{ ++i; } when it's not possible to overflow to save gas.

### Change proposed

```diff
-	for (uint256 x = temp.currentId + 1; x <= newId; x++) {
+	for (uint256 x = temp.currentId + 1; x <= newId; ) {
		nft.mint(msg.sender, x);
+       unchecked{x++;}
    }
```

### PoC
Original code
Running 14 tests for test/LPDA.t.sol:LPDATest
[PASS] test_Buy() (gas: 381183)

Code after apply changes
Running 14 tests for test/LPDA.t.sol:LPDATest
[PASS] test_Buy() (gas: 381124)

In the example of execute the test_BUY without and with the change proposed it's possible to have **59 gas saved** for instance.

### Lines in the code
[FixedPrice.sol#L65](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65)
[OpenEdition.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66)
[LPDA.sol#L73](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73)
