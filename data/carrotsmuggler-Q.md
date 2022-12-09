# 1. Missing zero Address check in sale receivers
`LPDAFactory.sol:31` checks for zero address for the sale.saleReceiver. This check is missing in the other two minters (FixedPrice, OpenEdition).

# 2. Consider adding an `endTime` to `FixedPrice` minter
`FixedPrice` minter does not finish the sale, or pay the creator until ALL the tokens specified have been minted. Also there is no way to stop the sale once started. If there has been an insufficient number of sales, only way for the creator to get paid for the tokens already minted is by minting the rest themselves and paying a 5% fee on it. Consider adding an `endTime` or a `forceCancel` option to conclude such sales.

# 3. Missing `require(amount!=0)` checks in minters
Add a `require(_amount!=0)` check in the minter to prevent emitting of events with 0 amounts.

# 4. Maximum value of `amount` in minters
Large values passed in `amount` can lead to out of gas error when minting a lot of NFTs. Consider adding a maximum cap to limit how many can be minted in a single transaction.

# 5. Transferring ownership of a sale contract keeps the sale.saleReceiver as last owner
Transferring out the ownership of the sale contract, or giving over admin roles in the Escher721 contract does not change the receiver of the proceeds of the sale. Consider sending the proceeds to the owner of the sale contract instead.

# 6. Incorrect comments in minters
All minters have comments referring to themselves as "fixed price sale". Example- LPDA.sol:56. Consider changing comments to accurately reflect the functionality.

# 7. No cap on royalty fees
In `Escher721.sol: setDefaulttRoyalty(address, uint96 feeNumerator)`, the value of `feeNumerator` is never checked/limited. So artists can set their royalty all the way up to 10000 (100%).