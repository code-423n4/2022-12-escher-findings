[1] Using block.timestamp/block block.number as arguments. dont use local variables.
file: uris/generative.sol:24
gas saved: 28

```diff
    function setSeedBase() external onlyOwner {
        require(seedBase == bytes32(0), "SEED BASE SET");

-        uint256 time = block.timestamp;
-        uint256 numb = block.number;
-        seedBase = keccak256(abi.encodePacked(numb, blockhash(numb - 1), time, (time % 200) + 1));
+        seedBase = keccak256(abi.encodePacked(block.number, blockhash(numb - 1), block.timestamp, (block.timestamp % 200) + 1));
    }
```

________________
[2] Using memory variable instead storage and move variable declaration after require
file: minters/openEditional.sol  function buy
2.1 uint24 amount = uint24(_amount);  move to place, after require(block.timestamp < temp.endTime, "TOO LATE");
2.2 IEscher721 nft = IEscher721(temp.edition);  move to place, after uint24 newId = amount + temp.currentId;
2.3 in require 
```
require(amount * sale.price == msg.value, "WRONG PRICE");
```
we can use temp variable and don't read form storage. sale.price -> temp.price
2.4 in for loop, move increment into body of for, and put it inside unchecked.


Final version of buy function
```
  function buy(uint256 _amount) external payable {
        Sale memory temp = sale;
        require(block.timestamp >= temp.startTime, "TOO SOON");
        require(block.timestamp < temp.endTime, "TOO LATE");
        uint24 amount = uint24(_amount);
        require(amount * temp.price == msg.value, "WRONG PRICE");
        uint24 newId = amount + temp.currentId;
        IEscher721 nft = IEscher721(temp.edition);

        for (uint24 x = temp.currentId + 1; x <= newId; ) {
            unchecked {
                x +=1;
            }
            nft.mint(msg.sender, x);
        }
        sale.currentId = newId;
        emit Buy(msg.sender, amount, msg.value, temp);
    }
```

Minumal Method Call. Gas Saved: +3
Average Method Call. Gas Saved: -124
Maximum Method Call. Gas Saved: -166

_____
[3] dont need set variable in default value
file: minters/LPDA.sol
Before
```
- uint48 public amountSold = 0;
+ uint48 public amountSold;
```
Deploy saved gas: 2218
______
[4] x = x + y is cheaper than x+=y and move variables declarations after require
file: minters/LPDA.sol  function buy
final version of buy
```
 function buy(uint256 _amount) external payable {
        Sale memory temp = sale;
        require(block.timestamp >= temp.startTime, "TOO SOON");
        uint256 price = getPrice();
        uint48 amount = uint48(_amount);
        require(msg.value >= amount * price, "WRONG PRICE");

        amountSold = amountSold + amount;
        uint48 newId = amount + temp.currentId;
        require(newId <= temp.finalId, "TOO MANY");
        IEscher721 nft = IEscher721(temp.edition);
-       receipts[msg.sender].amount += amount;
-        receipts[msg.sender].balance += uint80(msg.value);
+        receipts[msg.sender].amount = receipts[msg.sender].amount + amount;
+        receipts[msg.sender].balance = receipts[msg.sender].balance + uint80(msg.value);

        for (uint256 x = temp.currentId + 1; x <= newId; ) {
            unchecked {
                x++;
            }
            nft.mint(msg.sender, x);
        }

        sale.currentId = newId;

        emit Buy(msg.sender, amount, msg.value, temp);

        if (newId == temp.finalId) {
            sale.finalPrice = uint80(price);
            uint256 totalSale = price * amountSold;
            uint256 fee = totalSale / 20;
            ISaleFactory(factory).feeReceiver().transfer(fee);
            temp.saleReceiver.transfer(totalSale - fee);
            _end();
        }
    }

```
Minumal Method Call. Gas Saved: +6
Average Method Call. Gas Saved: -301
Maximum Method Call. Gas Saved: -714
