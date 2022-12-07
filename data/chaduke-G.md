G1. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L32
Passing ``_name`` and ``symbol`` as calldata can save gas.

G2. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L96
NO need to provide this function to override as it does not introduce new implementation. Deleting this function can save deployment gas.

G3. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L84
NO need to provide this function to override as it does not introduce new implementation. Deleting this function can save deployment gas.

G4. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L17
precomputing the keccak256 hash can save gas

G5. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L19
precomputing the keccak256 hash can save gas

G6. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L27
Passing ``_name`` and ``symbol`` as calldata can save gas.

G7. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L21
changing ``_newuri`` to calldata can save gas.

G8. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L70-L71
changing the two statements to the following to save gas: 
```
        receipts[msg.sender].amount = receipts[msg.sender].amount +amount;
        receipts[msg.sender].balance =receipts[msg.sender].balance + uint80(msg.value);
```

G8. https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L146
Simpying using ``emit End(sale);`` instead of ``end()`` will save gas.

G9: 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L66
Changing it to the following line can save gas:
```
 amountSold = amountSold+amount;
```
