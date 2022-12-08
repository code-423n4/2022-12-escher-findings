## L-01 - Event will emit the wrong value when a explicit downcast happens

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L57-L74

As explained in one of my reported issue, input of `uint256 _amount` being explicitly downcasted to a `uint48` here will cause issues. One of the side effects is that event emitted will be a `uint256` value, but the amount that is minted will be much smaller than that. This can create confusion for indexers. Fixing that issue would fix this too by setting `_amount = uint48(_amount)` right at the start.

```solidity
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

## L-02 -  setDefaultRoyalty can be changed anytime with no limits to it.

This makes it difficult for royalties receivers to trust the artist, since royalties amount can be change anytime to 0. Artist can get all the profits to himself. I recommend to allow royalties to be set only once.

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L64-L69

```
    function setDefaultRoyalty(
        address receiver,
        uint96 feeNumerator
    ) public onlyRole(DEFAULT_ADMIN_ROLE) {
        _setDefaultRoyalty(receiver, feeNumerator);
    }
```

## L-03 - Depreciating-soon selfdestruct is used to transfer funds to seller after sale ends.

The selfdestruct function is being used in `OpenEdition.sol` and `FixedPrice.sol` to transfer funds to users. This is not recommended as future forks of Ethereum is likely going to change that. See [EIP-4758](https://eips.ethereum.org/EIPS/eip-4758). Once removed, it will break the usage of it.

I recommend using a normal transfer instead of `selfdestruct` to transfer funds to users.

## L-04 - Creating a sale in FixedPrice factory and OpenEdition factory have very little input checks

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L29
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L29

It is lacking 2 important 0 checks in both of them. `saleReceiver()` address can be set to 0. This will cause the proceeds of sales to go to the 0 address, which are essentially lost. The variable `startPrice` can also be set to 0. Since default value in solidity is 0, sale creators may accidentally create a 0 priced sale, which is unlikely to be something that is wanted, and have all their NFTs be sold for free.

I recommend adding this 2 input checks.