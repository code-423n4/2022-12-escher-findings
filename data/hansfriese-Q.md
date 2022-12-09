### [L-01] Check address(0) for sale/fee receivers
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L29
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L42
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L29
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L43

Currently, it doesn't check address(0) for sale/fee receivers and the funds might be lost.

### [L-02] Wrong comments
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L56
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L56
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L91
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L25

These comments contain `fixed price sale/edition` for LPDA.

### [L-03] Users might lose funds with the unsafe cast.
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L62

```solidity
    require(_amount * sale_.price == msg.value, "WRONG PRICE");
    uint48 newId = uint48(_amount) + sale_.currentId; //@audit-info cast
```

Currently, it casts `_amount` after comparing with `msg.value` so users might lose their funds if `_amount > type(uint48).max`.

I submit it as a low risk as it's not so realistic unless users don't know the correct unit of `_amount`.

We should modify like below.

```solidity
    uint48 amount = uint48(_amount);
    require(amount * uint256(sale_.price) == msg.value, "WRONG PRICE");
    uint48 newId = amount + sale_.currentId;
```

### [L-04] Possible lost `msg.value`
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L71

```solidity
    require(msg.value >= amount * price, "WRONG PRICE");

    amountSold += amount;
    uint48 newId = amount + temp.currentId;
    require(newId <= temp.finalId, "TOO MANY");

    receipts[msg.sender].amount += amount;
    receipts[msg.sender].balance += uint80(msg.value); //@audit-info lost msg.value
```

Currently, it works if `msg.value >= amount * price` so `msg.value` might be greater than type(uint80).max.

In this case, users will lose the funds because the casted value is added to `receipts[msg.sender].balance`.

We should check if `msg.value < type(uint80).max` inside the function.

### [L-05] Wrong event for sale info
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L71
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L79

```solidity
    sale.currentId = newId;

    emit Buy(msg.sender, amount, msg.value, temp); //@audit-info wrong temp
```

`sale.currentId` was changed but it emits the original `temp` so that `currentId` will be wrong inside the event.

### [L-06] `OpenEdition.available()` returns a meaningless limit
- https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L116-L118

```solidity
    function available() public view returns (uint256) {
        return sale.endTime > block.timestamp ? type(uint24).max : 0; //@audit-info incorrect limit
    }
```

The real limit is `type(uint24).max - sale.currentId` and it would be sensitive when `sale.currentId` is near to `type(uint24).max`.