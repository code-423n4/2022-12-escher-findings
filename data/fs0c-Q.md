## [QA-01] No checks if `feeReceiver` is set to 0x0 in `setFeeReceiver` which may lead to fund loss.

### Impact

No checks if `feeReceiver` is set to 0x0 in `setFeeReceiver`  in all factory contracts which may lead to fund loss.

### POC

[https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42)

[https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42)

[https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46)

```solidity
function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
    }
```

### Recommended Mitigation Steps

Revert if `fees` variable is 0x0 in `setFeeReceiver` function.

```solidity
function setFeeReceiver(address payable fees) public onlyOwner {
				require( address(fees) != address(0x0))
        feeReceiver = fees;
    }
```