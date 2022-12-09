Consider using constructed variable instead of msg.sender. In this LPDA.SOL it is already constructed "factory"

[LPDA.SOL#L70](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L70)

## Mitigation Steps
 receipts[factory].amount += amount;