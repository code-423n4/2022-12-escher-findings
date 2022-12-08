1.
Title: Expression for `constant` values such as a call to `keccak256()`, should use `immutable` rather than `constant`

Proof of Concept:
[Escher.sol#L11-L13](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L11-L13)

Recommended Mitigation Steps:
Change from `constant` to `immutable`
reference: [here](https://github.com/ethereum/solidity/issues/9232)
________________________________________________________________________

2.
Title: empty `constructor`

Proof of Concept:
[Escher721.sol#L25](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L25)

Recommended Mitigation Steps:
Remove if unused for gas saving
________________________________________________________________________