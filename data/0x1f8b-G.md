
- [Gas](#gas)
    - [**1. Avoid compound assignment operator in state variables**](#1-avoid-compound-assignment-operator-in-state-variables)
        - [Total gas saved: **13 * 3 = 39**](#total-gas-saved-13--3--39)
    - [**2. Use the right type**](#2-use-the-right-type)

# Gas

## **1. Avoid compound assignment operator in state variables**

Using compound assignment operators for state variables (like `State += X` or `State -= X` ...) it's more expensive than using operator assignment (like `State = State + X` or `State = State - X` ...).

**Proof of concept (*without optimizations*):**

```javascript
pragma solidity 0.8.15;

contract TesterA {
uint256 private _a;
function testShort() public {
_a += 1;
}
}

contract TesterB {
Uint256 private _a;
function testLong() public {
_a = _a + 1;
}
}
```

Gas saving executing: **13 per entry**

```
TesterA.testShort: 43507
TesterB.testLong:  43494
```

**Affected source code:**

- [LPDA.sol:66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L66)
- [LPDA.sol:70](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L70)
- [LPDA.sol:71](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L71)

### Total gas saved: **13 * 3 = 39**

## **2. Use the right type**

Using the appropriate type in the methods mentioned below will avoid gas costs during the cast, in addition to avoiding possible overflow errors due to the insecure cast that is performed

```diff
-   function buy(uint256 _amount) external payable {
-       uint24 amount = uint24(_amount);
+   function buy(uint24 amount) external payable {
        Sale memory temp = sale;
        IEscher721 nft = IEscher721(temp.edition);
        require(block.timestamp >= temp.startTime, "TOO SOON");
        require(block.timestamp < temp.endTime, "TOO LATE");
        require(amount * sale.price == msg.value, "WRONG PRICE");
        uint24 newId = amount + temp.currentId;

        for (uint24 x = temp.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }
        sale.currentId = newId;

        emit Buy(msg.sender, amount, msg.value, temp);
    }
```

**Affected source code:**

- [OpenEdition.sol:58](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L58)
- [FixedPrice.sol:62](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L62)
- [LPDA.sol:59](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L59)
