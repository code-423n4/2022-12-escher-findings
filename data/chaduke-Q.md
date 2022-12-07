QA1. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L64
It might be better to change the line to
```
 require(msg.value == amount * price, "WRONG PRICE");
```

QA2: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L42
A zero address check is necessary, otherwise some fee will be permanently lost.

QA3: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L92
we should not use ``transfer()`` to send eth, instead, we should use .call.value(...)(""), according to consensys see
https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

QA4: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L84
It might be better to use a two-step procedure to transfer the ownership. If a wrong/invalid sale.saleReiver is input, the ownership will be lost forever. The two step procedure will first assign a pending owner, and then the pending owner needs to accept the owner. 

QA5: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L57
amount zero check needs to be performed

QA6: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L43
A zero address check needs to be performed for ``feeReceiver``, otherwise fee might be lost forever.

QA7: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L57
amount zero check needs to be performed

QA8: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L29
A zero address check needs to be performed for ``sale.saleReceiver``, otherwise owernship of the ``FixedPrice" contract will be transferred to a zero address owner and also, when a sale close, all money will be lost to the zero address.

QA9: https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L99
To avoid multiple refund transactions by the same user, it might be better to allow refund only after a sale ends.
```
function refund() public {
        require(block.timestamp > sale.endTime, "TOO early");

        Receipt memory r = receipts[msg.sender];
        uint80 price = uint80(getPrice()) * r.amount;
        uint80 owed = r.balance - price;
        require(owed > 0, "NOTHING TO REFUND");
        receipts[msg.sender].balance = price;
        payable(msg.sender).transfer(owed);
    }
```
