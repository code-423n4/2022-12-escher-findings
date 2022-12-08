## Issue 1:
When a sale is finished, `End` event is emitted. The same event is emitted when sale is cancelled. Therefore it can be unclear if sale actually finnished or it was cancelled. It would be clearer if there was a separate event called `Cancel` for example.

### Recommendation:
Add `Cancel` event to sales.


## Issue 2:
It is stated that Escher is a curated marketplace. This would mean that sales can be created only by authorized addresses. It can be seen in `Escher721Factory.sol` that only allowed addresses (those with CREATOR_ROLE) can create new Echer721 contracts. However, when actually creating sales in `FixedPriceFactory.sol` / `LPDAFactory.sol` / `OpenEditionFactory.sol` it is never checked if the `Echer721` contract was created through the factory. This means that anyone can deploy an `Echer721` contract (or any contract as long as it matches the interface) without using the factory and create a sale on the protocol anyway, even if he is not "creator".

### Recommendation:
You can add a `mapping(address => bool) public isClone` in `Echer721Factory` to keep track of all clones that are deployed through the factory. You can then call `Echer721Factory(factoryAddress).isClone(_sale.edition)` when creating a sale to check whether the contract was created using the factory.
Another option is to have front end only keep track of sales that are created through the factory.


## Issue 3:
In readme where sale process is described, it states that "If the artist would like sales and royalties to go somewhere other than the default royalty receiver, they must call `setTokenRoyalty`". However, all of the sale contracts ignore royalties. Creator can set who will receive the funds when he creates a sale by specifying `saleReceiver`. But `saleReceiver` receives all of the funds (`- protocolFee`). So there is no royalties mechanism in sales. Furthermore, there is no `setTokenRoyalty` in `Escher721`, but only `setDefaultRoyalty` function. It is possible that this is just a mistake in the description, but it can also be a functionality that is missing from the code.

### Recommendation:
Add `setTokenRoyalty` function in `Escher721` like this:
```
function setTokenRoyalty(
        uint256 tokenId,
        address receiver,
        uint96 feeNumerator
    ) public onlyRole(DEFAULT_ADMIN_ROLE) {
        __setTokenRoyalty(tokenId, receiver, feeNumerator);
    }
```

Implement royalties in sales contracts. Notice that if you enable tokens to have different royalties than the default royalties, you would have to calculate royalties for each token sold separately. So it is probably better to only enable default (contract wide) royalties in case you are implementing them in sales.
Here is an example how you can use royalties: 
```
uint256 fee = totalPrice / 20;

(address royaltyReceiver, uint256 royaltyAmount) = IERC2981(sale.edition).royaltyInfo(tokenId,totalPrice);

(bool paidFee, ) = ISaleFactory(factory).feeReceiver().call{value: fee}("");
require(paidFee, "Failed to pay fee");

(bool paidRoyalty, ) = sale.saleReceiver.call{value: royaltyAmount}(""); 
require(paidRoyalty, "Failed to pay royalty");

(bool paidout, ) = sale.saleReceiver.call{value: totalPrice - fee - royaltyAmount}(""); 
require(paidout, "Failed to pay");

```

## Issue 4:
There are multiple instances where `.transfer()` is used to transfer ether. `.transfer()` is dependent on gas cost, because it forwards a fixed amount of gas (2300). Gas specific code should be avoided, because gas costs can change and that can cause the transaction to fail.

### Instances:

https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/LPDA.sol#L85
https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/FixedPrice.sol#L109
https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/OpenEdition.sol#L92

### Recommendation:
Use `call` instead and check if transfer is successful.
```
(bool sent, ) = receiver.call{value: ethAmount}(""); 
require(sent, "Failed to send Ether");
```
Notice: When you transfer ether and the receiver is a contract, its `fallback()` function is executed. And using `call` without limiting the amount of gas, it is able to do complex operations. This can cost you more gas, but more importantly it can allow re-entrancy attacks. So make sure you are following Check - Effect - Interaction pattern and in cases where that is not enough include `nonReentrant` modifier.