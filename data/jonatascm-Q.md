# Missing validation can lead to loss of fees

## Target codebase

[FixedPriceFactory.sol#L42-L44](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42-L44)

[OpenEditionFactory.sol#L42-L44](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42-L44)

[LPDAFactory.sol#L46-L48](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46-L48)

## Vulnerability details

Missing validation of `fees` in `setFeeReceiver` function lead to loss of fees when closing a sale.

## Recommended Mitigation Steps

Is recommended to validate if `fees` param is not address(0)

```diff
function setFeeReceiver(address payable fees) public onlyOwner {
	require(fees != address(0), "INVALID FEE RECEIVER");
  feeReceiver = fees;
}
```

# Lack of validations when creating sales

## Target codebase

[FixedPriceFactory.sol#L29-L38](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L29-L38)

[OpenEditionFactory.sol#L29-L38](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L29-L38)

## Vulnerability details

Missing validation of `prices` when creating a sale, enabling the `CREATOR` to create a sale with zero price. In `LPDAFactory` should also check the `finalPrice` param

## Recommended Mitigation Steps

Is recommended to validate if `prices` and `finalPrice` params is greater than 0

---

# Change ownership should be two step process

## Target codebase

[FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)

[OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

[LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

## Vulnerability details

All contracts are inherited from OpenZeppelin's Ownable and OwnableUpgradable contract which enables the onlyOwner role to transfer ownership to another address. It's possible that the onlyOwner role mistakenly transfers ownership to the wrong address, resulting in a loss of the onlyOwner role

## Recommended Mitigation Steps

Is recommended to implement two step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

---

# Buy event is emitted with not updated sale

## Target codebase

[FixedPrice.sol#L71](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L71)

[LPDA.sol#L79](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L79)

## Vulnerability details

The `Buy` event is emitted with temp variable which is not updated, if any service is getting this event will be verifying an invalid variable.

## Recommended Mitigation Steps

Is recommended to emit the event with `sale` instead of `temp`