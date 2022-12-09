## Missing check on LPDA sale final price

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L29-L42

```solidity
    function createLPDASale(LPDA.Sale calldata sale) external returns (address clone) {
        require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
        require(sale.saleReceiver != address(0), "INVALID SALE RECEIVER");
        require(sale.startTime >= block.timestamp, "INVALID START TIME");
        require(sale.endTime > sale.startTime, "INVALID END TIME");
        require(sale.finalId > sale.currentId, "INVALID FINAL ID");
        require(sale.startPrice > 0, "INVALID START PRICE");
        require(sale.dropPerSecond > 0, "INVALID DROP PER SECOND");

        clone = implementation.clone();
        LPDA(clone).initialize(sale);

        emit NewLPDAContract(msg.sender, sale.edition, clone, sale);
    }
```

It should check whether finalPrice <= startPrice or not by adding this line

```solidity
require(sale.finalPrice <= sale.startPrice, "INVALID FINAL PRICE");
```

## lowestPrice can be negative

```solidity
    function lowestPrice() public view returns (uint256) {
        return sale.startPrice - (sale.dropPerSecond * (sale.endTime - sale.startTime));
    }
```

Assume startPrice = 5, dropPerSecond = 1, sale.endTime - sale.startTime = 10

`sale.startPrice - (sale.dropPerSecond * (sale.endTime - sale.startTime))` = 5 - (1 * 10) = 5 - 10 = -5 which is negative

As a result, lowestPrice will always revert in such cases

`sale.startPrice - (sale.dropPerSecond * (sale.endTime - sale.startTime))` must be truncated to zero if negative and compared to `sale.finalPrice` if it is greater than zero

## Don't use payable(...).transfer(...)

Using deprecated `transfer()` on address payable may revert in these cases:

1. The withdraw recipient is a smart contract that does not implement a payable function.
2. The withdraw recipient is a smart contract that does implement a payable fallback which uses more than 2300 gas unit.
3. The withdraw recipient is a smart contract that implements a payable fallback function that needs less than 2300 gas units but is called through proxy, raising the call's gas usage above 2300.

Additionally, using higher than 2300 gas might be mandatory for some multisig wallets.

Consider using these

```solidity
      (bool success, ) = payAddress.call{value: payAmt}("");
      require(success, "ETH transfer failed");
```