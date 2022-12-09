## Summary
### Low Risk
|      | Issue                                                                            |
|------|----------------------------------------------------------------------------------|
| L-01 | Use of native transfer function is not recommended                               |
| L-02 | Unsafe cast in `FixedPrice.buy()`                                                |
| L-03 | Wrong error string in `Escher._grantRole` for any other role than `CREATOR_ROLE` |
| L-04 | Remove unused code                                                               |


## Low
### [L‑01] Use of native transfer function is not recommended
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L109
The protocol contracts use the native `transfer` function to transfer ETH to the `feeReceiver` or the `creator`.

This function will revert if the recipient is a smart contract and its fallback function uses more than 2300 gas.

To avoid any issue with a careless `creator`, prefer to use `call` to send ETH.

### [L‑02] Unsafe cast in `FixedPrice.buy()`


While mathematical operations revert on overflow/underflow in Solidity, this is not the case when casting a `uint256`.

```solidity
57:     function buy(uint256 _amount) external payable {
58:         Sale memory sale_ = sale;
59:         IEscher721 nft = IEscher721(sale_.edition);
60:         require(block.timestamp >= sale_.startTime, "TOO SOON");
61:         require(_amount * sale_.price == msg.value, "WRONG PRICE");
62:         uint48 newId = uint48(_amount) + sale_.currentId; // @audit - can overflow
```

If the buyer supplies a `_amount` greater than `type(uint48).max`, `uint48(_amount)` will overflow.

The issue is that at this line, the user will have paid the price of `_amount` of tokens, while only receiving a much lower number of NFTs.

As `type(uint48).max== 281474976710655`, it is however very unlikely to see this happening - unless the collection has a very high number of tokens.

### Recommendation

`buy()` should take a `uint48` input.

```diff
-57:  function buy(uint256 _amount) external payable
+57:  function buy(uint48 _amount) external payable
```

### [L‑03] Wrong error string in `Escher._grantRole` for any other role than `CREATOR_ROLE`

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L61
```solidity
61:         require(balanceOf(_account, uint256(_role)) == 0, "Already Creator"); //@audit - wrong error if calling for other role
```

`_grantRole` reverts with an "Already Creator" error, which is incorrect if called for another `_role`, such as `CREATOR_ROLE` or `MINTER_ROLE`.

### Recommendation

Use a revert string matching the role being added - as it is done in the OpenZeppelin `AccessControl` implementation

```solidity
if (balanceOf(_account, uint256(_role)) != 0) {
    revert(string(abi.encodePacked(
        "Already ",
        Strings.toHexString(uint256(_role), 32);
}
```

### [L‑04] Remove unused code

This function is not called by any contract in the project

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L27
```solidity
27:     function tokenToSeed(uint256 _tokenId) internal view returns (bytes32)
```

Either remove it, or consider changing the function visibility to `external`



