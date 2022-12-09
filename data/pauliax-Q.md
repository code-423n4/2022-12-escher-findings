* In ```LPDA``` and ```FixedPrice``` users can still buy even after the ```endTime```. The check is missing:
```solidity
  require(block.timestamp < temp.endTime, "TOO LATE");
```

* Not sure if this is intended or a bug but NFT id of 0 is skipped:
```solidity
  for (uint48 x = sale_.currentId + 1; x <= newId; x++)
```
And overall it always starts minting from the next id.

* Contracts contain several unsafe castings, e.g.:
```solidity
  uint48 newId = uint48(_amount) + sale_.currentId;
  receipts[msg.sender].balance += uint80(msg.value);
  uint80 price = uint80(getPrice()) * r.amount;
```
This is usually not a problem because it will not be possible to mint that many NFTs or send so much ETH anyway, but you still might consider using ```SafeCast``` library.

* When creating the sales in factories, it validates the user role:
```solidity
  require(IEscher721(_sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
```
However, the address of an ```edition``` itself is not validated, so a user may pass an arbitrary address, a custom contract that always returns true for the ```hasRole```. Later this contract is invoked when minting the NFTs basically giving the control back to the owner. At this point the owner might decide to do something nasty, e.g. revert for certain users, not mint anything at all, or re-enter the contract and bypass the ```finalId``` limit because the ```currentId``` is updated only after the minting:
```solidity
  for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
      nft.mint(msg.sender, x);
  }

  sale.currentId = newId;
```
I am not entirely sure about the severity of this problem, so submitting it in a QA report because I think that the owners mainly do damage themselves this way unless they use an NFT contract from someone else they trusted but apparently it was a malicious contract. You should still consider adding a whitelist of possible ```edition``` contracts. 