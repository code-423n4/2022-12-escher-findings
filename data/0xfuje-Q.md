
# Low issues

## 1 - Insufficient validation for _amount in buy functions of auction contracts

**Target**: buy() in [FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol), [LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol), [OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

**Problem 1**: 
buy() function doesn't validate that _amount is 0 passed and neither if msg.value is 0. That means a malicious attacker can call the function with 0 _amount and 0 msg.value and the function will emit a Buy() event every time.

The consequence is that front-end / UI could get clouded with a bunch of wrong Buy() events.

**Problem 2**:
buy() function could overflow if a high enough _amount is passed in because it's always converted down to a lower uint. Although in normal scenarios it's not likely a user would pass in such a high _amount it leads to the function not working as expected.
  
**Recommendation:**
Validate if the passed in amount is other than zero (in all the buy functions), only than let the function continue execution ( require(amount != 0) ). Consider protecting against overflow (e.g. make passed in _amount uint24 or other lower uint so doesn't need to be converted).

## 2 - Insufficient validation of create sale functions can allow an attack on front-end via creating a bunch of fake sale contracts
**Target**: createFixedSale() in [FixedPriceFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol)
createLPDASale() in [LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol)
createOpenEdition() in [OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)

Malicious attacker can create a bunch of fake sale contracts by calling either createFixedSale(), createLPDASale() or createOpenEdition() functions and cloud the front-end.

**Recommendation:**
Use stricter validation on the sale contract creation or validate the correct contracts in the front-end.

## 3 - Possible DoS because incorrect _creator passed in initialize() of Escher721.sol
**Target**: Escher721.sol - [L32-L46](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L32-L46)

The initialize() function grants the roles of DEFAULT_ADMIN_ROLE, MINTER_ROLE and URI_SETTER_ROLE to the _creator address passed in to the initialization. If the zero address is accidentally passed as a _creator all these roles are lost and the creator can't call mint(), updateURIDelegate(), setDefaultRoyalty(), resetDefaultRoyalty() functions making the contract unusable.

**Recommended Mitigation Steps**:
Consider adding a  zero address check for the _creator address in intialize()

## 4 - Lack of data validation for _escher in constructor of Escher721Factory.sol
**Target**: Escher721Factory.sol - [L17-L24](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol#L17-L24)

Missing zero check for the passed in _escher address parameter can lead to the wrong address passed in that makes Escher unusable. 

**Recommended Mitigation Steps**:
Consider adding a zero address check and contract validation (e.g. by checking extcodesize) of _escher address.

## 5 - Front running danger across the contracts
**Note**: Sorry, didn't had time to investigate this more
Most of the contracts have an initialize function that could be front-run

# Non-Critical issues

## 1 - Provide more descriptive revert
**Target**: Escher.sol - [L45](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L45), [L56](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L56)

"Soulbound" doesn't entirely describe why the transfer reverts if one has no context of soulbound tokens. Consider using a more descriptive revert that explains why soulbound tokens can't be transferred by these transfer functions.

## 2 - Inconsistency in require() reverts
**Target**: Escher.sol - [L61](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L61)

The require() statements through the contracts all use "CAPITALIZED_ERRORS", but in Escher.sol the _grantRole reverts with "Already Creator". Using "ALREADY CREATOR" would be more consistent.

## 3 - Document feeNumerator is in basis points
**Target**: Escher721.sol - [L63](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L63)

It's not clear at first look how feeNumerator is scaled in setDefaultRoyalty() of Escher721.sol. Document that feeNumerator is in fact calculated in basis points (because it's from openzeppelin's ERC2981 implementation that uses basis points) so it's easier to understand the function/parameter.

## 4 - Inconsistent _end() functions in auction contracts
**Target**: _end() in [FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol), [LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol), [OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

**FixedPrice.sol**: the internal _end() function emits an End() event than calculates the fee and transfer it to the feeReceiver than selfdestructs the contract to the saleReceiver

**LPDA.sol**: _end() function only emits an event, fee calculation is done outside of the function

**OpenEdition.sol**: _end() function emits an event than selfDestructs to the saleReceiver, fee calculation and transfer is done outside of the function

If all these _end() functions would be similar it would be better practice because code would look more clean. One possible refactoring is removing the _end() in LPDA.sol and emitting the End() event in the buy() if statement where (newId == temp.finalId) OR moving the entire block inside the (newId == temp.finalId)  if statement to _end().

## 5 - Wrong assumption that msg.sender is trusted actor aka feeReceiver
**Target**: constructor of [FixedPriceFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol), [LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol), [OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)

The constructors of the minter factory contracts all assume that feeReceiver will be the deployer aka msg.sender of the factory contracts. This assumptions could be inaccurate since in theory anyone could deploy the factory contracts.

**Recommended Mitigation Steps:**
Consider setting the feeReceiver via an address parameter passed in the constructor so it gives more control of setting the feeReceiver and is less prone to unexpected outcomes.

## 6 - fees of setFeeReceiver should be more descriptive parameter
**Target**: setFeeReceiver of [FixedPriceFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol), [LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol), [OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)

fees doesn't exactly describe that it's the fee receiver address. Consider renaming it to a more descriptive parameter for example _feeReceiver. 

## 7- Incorrect comments: nextId and reaminingSupply
**Target**:  FixedPrice.sol - [L12-13](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L12-L13)

These two commented lines are about nextId and remainingSupply. It's not clear where these variables can be found. Review these lines and either implement nextId and remainingSupply or delete the lines.

## 8 - Consider two-step ownership change for the auction contracts
**Target**: [FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol), [LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol), [OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

The inherited OwnableUpgradeable contract does not support two-step ownership change of the auction contracts. Consider implementing it because of the added safety it provides.

## 9 - Undescriptive variables in LPDA.sol constructor
**Target:** LPDA.sol - [L49-53](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L49-L53)

It's hard to understand what these variables are for without looking up the Sale struct, consider renaming them. Also other auction constructors doesn't have variables initialized in them, they are just passed into sale: consider doing the same here for consistency.

## 10 - Consider two-step transfer or a zero address check for setFeeReceiver() in the minter factory contracts
**Target**:  setFeeReceiver() of [FixedPriceFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol), [LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol), [OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)

The setFeeReceiver can be mistakenly set to the zero address or to the wrong fee receiver because of a dusting attack on the owner. If it remains undetected, when a sale ends the fees are sent to the wrong fee receiver.

**Recommended Mitigation Steps**:
Consider adding a zero address and a two-step transfer of setFeeReceiver where first fee receiver is nominated and than accepted. This would make setting feeReceiver less error prone.

## 11 - Using selfdestruct is considered bad practice
**Note**: Sorry, didn't had time to elaborate on this more