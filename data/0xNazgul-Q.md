## [NAZ-L1] Missing Equivalence Checks in Setters
**Severity**: Low
**Context**: [`Base.sol#L10`](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L10), [`Generative.sol#L14`](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L14), [`FixedPriceFactory.sol#L42`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42), [`OpenEditionFactory.sol#L42`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42), [`LPDAFactory.sol#L46`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46), [`Escher.sol#L21`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L21), [`Escher.sol#L27`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L27)

**Description**:
Setter functions are missing checks to validate if the new value being set is the same as the current value already set in the contract. Such checks will showcase mismatches between on-chain and off-chain states.

**Recommendation**:
This may hinder detecting discrepancies between on-chain and off-chain states leading to flawed assumptions of on-chain state and protocol behavior.


## [NAZ-L2] Missing Time locks
**Severity**: Low 
**Context**: [`Base.sol#L10`](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L10), [`Generative.sol#L14`](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L14), [`Generative.sol#L19`](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L19), [`FixedPriceFactory.sol#L42`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42), [`OpenEditionFactory.sol#L42`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42), [`LPDAFactory.sol#L46`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46), [`Escher.sol#L21`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L21), [`Escher.sol#L27`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L27), [`Escher721.sol#L57`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L57), [`Escher721.sol#L64`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L64), [`Escher721.sol#L72`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L72)

**Description**:
When critical parameters of systems need to be changed, it is required to broadcast the change via event emission and recommended to enforce the changes after a time-delay. This is to allow system users to be aware of such critical changes and give them an opportunity to exit or adjust their engagement with the system accordingly. None of the onlyOwner functions that change critical protocol addresses/parameters have a timelock for a time-delayed change to alert: (1) users and give them a chance to engage/exit protocol if they are not agreeable to the changes (2) team in case of compromised owner(s) and give them a chance to perform incident response.

**Recommendation**:
Users may be surprised when critical parameters are changed. Furthermore, it can erode users' trust since they can’t be sure the protocol rules won’t be changed later on. Compromised owner keys may be used to change protocol addresses/parameters to benefit attackers. Without a time-delay, authorized owners have no time for any planned incident response.


## [NAZ-L3] Missing Zero-address Validation
**Severity**: Low
**Context**: [`FixedPriceFactory.sol#L17`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L17), [`FixedPriceFactory.sol#L42`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42), [`OpenEditionFactory.sol#L42`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42), [`LPDAFactory.sol#L46`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46)

**Description**:
Lack of zero-address validation on address parameters may lead to transaction reverts, waste gas, require resubmission of transactions and may even force contract redeployments in certain cases within the protocol.

**Recommendation**:
Consider adding explicit zero-address validation on input parameters of address type.


## [NAZ-L4] Lack of Event Emission For Critical Functions
**Severity**: Low
**Context**: [`Base.sol#L10`](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L10), [`Generative.sol#L14`](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L14), [`Generative.sol#L19`](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L19), [`FixedPriceFactory.sol#L42`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42), [`OpenEditionFactory.sol#L42`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42), [`LPDAFactory.sol#L46`](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46)

**Description**:
Several functions update critical parameters that are missing event emission. These should be performed to ensure tracking of changes of such critical parameters.

**Recommendation**:
Consider adding events to functions that change critical parameters.


## [NAZ-N1] Code Contains Empty Blocks
**Severity**: Informational
**Context**: [`Escher721.sol#L25`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L25)

**Description**:
It's best practice that when there is an empty block, to add a comment in the block explaining why it's empty.

**Recommendation**:
Consider adding `/* Comment on why */` to the empty blocks.


## [NAZ-N2] Function && Variable Naming Convention
**Severity** Informational
**Context**: [`Generative.sol#L27`](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L27)

**Description**:
The linked variables do not conform to the standard naming convention of Solidity whereby functions and variable names(local and state) utilize the `mixedCase` format unless variables are declared as `constant` in which case they utilize the `UPPER_CASE_WITH_UNDERSCORES` format. Private variables and functions should lead with an `_underscore`.

**Recommendation**:
Consider naming conventions utilized by the linked statements are adjusted to reflect the correct type of declaration according to the [Solidity style guide](https://docs.soliditylang.org/en/latest/style-guide.html). 


## [NAZ-N3] Spelling Errors
**Severity**: Informational
**Context**: [`Escher.sol#L8 (editionized => ?I don't know?)`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L8), [`Escher721.sol#L18 (editionized => ?I don't know?)`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L18), [`Escher721.sol#L18 (art work => artwork)`](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L18)

**Description**:
Spelling errors in comments can cause confusion to both users and developers.

**Recommendation**:
Consider checking all misspellings to ensure they are corrected.


## [NAZ-N4] Missing or Incomplete NatSpec
**Severity**: Informational
**Context**: [`All Contracts`](https://github.com/code-423n4/2022-12-escher/tree/main/src)

**Description**:
Some functions are missing @notice/@dev NatSpec comments for the function, @param for all/some of their parameters and @return for return values. Given that NatSpec is an important part of code documentation, this affects code comprehension, auditability and usability.

**Recommendation**:
Consider adding in full NatSpec comments for all functions to have complete code documentation for future use

## [NAZ-N5] Floating Pragma
**Severity**: Informational
**Context**: [`All Contracts`](https://github.com/code-423n4/2022-12-escher/tree/main/src)

**Description**:
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

**Recommendation**: 
Consider locking the pragma version.