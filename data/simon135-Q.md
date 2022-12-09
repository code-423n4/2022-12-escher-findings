##### no address zero checks
 put an address zero check on the sale receiver  in `fixedPriceFactory.sol`

##### change var to make it more readable
```solidity 
function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
    }
```
make instead: receiver 
##### `cancel` function in `FixedPrice.sol`   should fix the require check 
The   require check 

```solidity 

     require(block.timestamp < sale.startTime, "TOO LATE");
```
can make the owner wait and can cause the sale receiver a loss of items 
because if the `startTime=block.timestamp` is in the same block the owner won't be able to cancel.
ex:
Alice makes a sale but then a second later makes a tx to cancel the function will revert 
and then someone cant buy an nft when Alice didn't them sold.
instead, make it:
```solidity 
require(block.timestamp<=sale.startTime)
```
##### reentrancy risk because of checks and effects not followed
the risk is very low because the mint doesn't have an external call to an untrusted contract
but if in the future of the protocol the `nft` contract changes  then there will be a reentrancy risk 
```solidity 
    for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }

        sale.currentId = newId;
```
an attacker gets more tokens without an update to the  `currentId` 
instead:
do checks and effects pattern 
#### don't use `selfdestruct`  because it can brick the contract
instead, use a call even though it costs more  gas 
##### the logic for the  `for` loop is not very readable 
```
(uint48 x = sale_.currentId + 1; x <= newId; x++)
```
ex: users buy 10 tokens 
they get 1,10 
and the 10 token 
instead, make it like a regular `for` loop 
instead:
```
(uint48 x = sale_.currentId ; x < newId; x++)
```

##### with a high number of tokens or starting off with a high `currentId` the function will revert
poc:
``
```solidity 
  function test_overflow() public payable {
        // make the fixed-price sale
        FixedPrice.Sale memory s;
        FixedPrice other;
        s = FixedPrice.Sale({
            currentId: type(uint48).max - 1,
            finalId: type(uint48).max,
            edition: address(edition),
            price: uint96(uint256(1)),
            saleReceiver: payable(address(6)),
            startTime: uint96(block.timestamp)
        });
        other = FixedPrice(fixedSales.createFixedSale(s));
        // authorize the fixed price sale to mint tokens
        edition.grantRole(edition.MINTER_ROLE(), address(other));
        vm.prank(address(6));
        uint256 b = uint256(type(uint48).max) + 1;
        vm.deal(address(6), b);
        other.buy{value: 1}(1);
    }
```