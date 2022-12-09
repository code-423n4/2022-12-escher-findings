## No way to reduce endTime

Contract:
https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L91

Impact:
If the seller has set a very high endTime by mistake then the sale would never end resulting in seller in never receiving the collected funds

Steps:
1. Observe that `OpenEdition` contract does not allow user to change the sale.endTime 
2. This becomes a problem if seller has mistakenly set endTime to a really high value
3. Since the sale cannot be finalized till endTime is reached

```
function finalize() public {
        Sale memory temp = sale;
        require(block.timestamp >= temp.endTime, "TOO SOON");
        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
        _end(temp);
    }
```


Recommendation:
Add a new function which allows user to change endTime