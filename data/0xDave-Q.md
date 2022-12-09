At line of 65 of `FixedPrice.sol`, NFT with `currentId` of 0 will not be minted because the `currentId` is always 0 in the beginning. The sale will end without minting if a tokenId of 0 is sold by the creator. Due to this, it is necessary to inform the creator in advance not to make NFT with a tokenId of 0.

```
        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }
```