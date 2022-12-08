It was found that setFeeReceiver() functionality does not emit an event. 


https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46-L49

File: /src/minters/LPDAFactory.sol#L46-L49

```

    function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
    }
}

````
