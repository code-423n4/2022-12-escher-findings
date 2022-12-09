## [L-01] `transfer` is used instead of `call`

transfer is used to transfer value, which could cause issue if the receiver is a contract.

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L86
```solidity
            temp.saleReceiver.transfer(totalSale - fee);
```

## [L-02] `available` is not calculated according to actual remaining nft at any point in time

The maximum amount of NFT is `type(uint24).max` however the `available` function does not take into account the already bought NFTs and always return `type(uint24).max` during the sale period.

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L116-L118
```solidity
    function available() public view returns (uint256) {
        return sale.endTime > block.timestamp ? type(uint24).max : 0;
    }
```