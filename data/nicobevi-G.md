# Copy just currentId to memory instead of the entire sale struct will save gas

## Where

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L57-L74

```solidity
    /// @notice buy from a fixed price sale after the sale starts
    /// @param _amount the amount of editions to buy
    function buy(uint256 _amount) external payable {
        Sale memory sale_ = sale;
        IEscher721 nft = IEscher721(sale_.edition);
        require(block.timestamp >= sale_.startTime, "TOO SOON");
        require(_amount * sale_.price == msg.value, "WRONG PRICE");
        uint48 newId = uint48(_amount) + sale_.currentId;
        require(newId <= sale_.finalId, "TOO MANY");

        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }

        sale.currentId = newId;

        emit Buy(msg.sender, _amount, msg.value, sale);

        if (newId == sale_.finalId) _end(sale);
    }
```

Replace the `Sale memory sale` definition and just copy `sale.currentId` to memory.

```solidity
    function buy(uint256 _amount) external payable {
        IEscher721 nft = IEscher721(sale.edition);
        require(block.timestamp >= sale.startTime, "TOO SOON");
        require(_amount * sale.price == msg.value, "WRONG PRICE");
        
        uint48 currentId_ = sale.currentId;
        
        uint48 newId = uint48(_amount) + currentId_;
        
        require(newId <= sale.finalId, "TOO MANY");


        for (uint48 x = currentId_ + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }

        sale.currentId = newId;

        emit Buy(msg.sender, _amount, msg.value, sale);

        if (newId == sale.finalId) _end(sale);
    }
```

# Unnecesary sale parameter on `_end()` call

Since there is just one sale and after it's ended the contract is selfdestructed, sale can be referenced from storage. Making unnecesary a sale parameter on `_end()` definition.

## Where
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L107-L112

```solidity
  function _end(Sale memory _sale) internal {
      emit End(_sale);
      ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
      selfdestruct(_sale.saleReceiver);
  }
```

Replace the code with
```solidity
  function _end() internal {
      Sale memory _sale = sale;
      ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
      selfdestruct(_sale.saleReceiver);
      emit End(_sale);
  }
```

## Only cache `sale.currentId` and `sale.finalId` in memory instead of the entire sale struct on `buy()` on `LPDA.sol` to save gas.

```solidity
  function buy(uint256 _amount) external payable {
      uint48 amount = uint48(_amount);
      
      // Sale memory temp = sale; // Remove this

      IEscher721 nft = IEscher721(sale.edition);
      require(block.timestamp >= sale.startTime, "TOO SOON");
      uint256 price = getPrice();
      require(msg.value >= amount * price, "WRONG PRICE");

      // Cache only this two values
      uint48 currentId_ = sale.currentId;
      uint48 finalId_ = sale.finalId;

      amountSold += amount;
      uint48 newId = amount + currentId_;
      require(newId <= finalId_, "TOO MANY");

      receipts[msg.sender].amount += amount;
      receipts[msg.sender].balance += uint80(msg.value);

      for (uint256 x = currentId_ + 1; x <= newId; x++) {
          nft.mint(msg.sender, x);
      }

      sale.currentId = newId;

      emit Buy(msg.sender, amount, msg.value, sale);

      if (newId == finalId_) {
          sale.finalPrice = uint80(price);
          uint256 totalSale = price * amountSold;
          uint256 fee = totalSale / 20;
          ISaleFactory(factory).feeReceiver().transfer(fee);
          sale.saleReceiver.transfer(totalSale - fee);
          _end();
      }
  }
```

## `amountSold` could be replaced with `sale.currentId + 1` on `LPDA.sol`.

Replace

```solidity
    // uint48 public amountSold = 0; // Replace this with a view function

    function amountSold() public view returns (uint48) {
        return sale.currentId;
    }
```

```solidity
    // LPDA.sol Line 66
    // amountSold += amount; // Comment this line

    // LPDA.sol Line 83
    // uint256 totalSale = price * amountSold; Change this line with
    uint256 totalSale = price * amountSold();
```

# Access the receipt object just once using the `storage` modifier will save some gas.

## Where
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L70-L71

```solidity
  // Instead of this
  // receipts[msg.sender].amount += amount;
  // receipts[msg.sender].balance += uint80(msg.value);

  // Do this
  Receipt storage receipt = receipts[msg.sender];
  receipt.amount += amount;
  receipt.balance += uint80(msg.value);
```

## Use unchecked on for loops

# Where
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73-L75

```solidity
  // Replace this
  // for (uint256 x = temp.currentId + 1; x <= newId; x++) {
  //    nft.mint(msg.sender, x);
  // }

  // With this
  for (uint256 x = temp.currentId + 1; x <= newId;) {
      nft.mint(msg.sender, x);
      unchecked {
        ++x;
      }
  }
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65-L67

```solidity
  // Replace this
  // for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
  //    nft.mint(msg.sender, x);
  // }

  // With this
  for (uint48 x = sale_.currentId + 1; x <= newId;) {
    nft.mint(msg.sender, x);
    unchecked {
      ++x;
    }
  }
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66-L68

```solidity
  // Replace this
  for (uint24 x = temp.currentId + 1; x <= newId; x++) {
      nft.mint(msg.sender, x);
  }

  // With this
  for (uint24 x = temp.currentId + 1; x <= newId;) {
      nft.mint(msg.sender, x);
      unchecked {
        ++x;
      }
  }
```

# Use `temp.price` instead of `sale.price` on `buy()`

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L63

```solidity
    function buy(uint256 _amount) external payable {
        uint24 amount = uint24(_amount);
        Sale memory temp = sale;
        IEscher721 nft = IEscher721(temp.edition);
        require(block.timestamp >= temp.startTime, "TOO SOON");
        require(block.timestamp < temp.endTime, "TOO LATE");
        // require(amount * sale.price == msg.value, "WRONG PRICE"); // Replace this line with the line below
        require(amount * temp.price == msg.value, "WRONG PRICE");
        uint24 newId = amount + temp.currentId;

        for (uint24 x = temp.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }
        sale.currentId = newId;

        emit Buy(msg.sender, amount, msg.value, temp);
    }
```