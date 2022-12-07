selfdestruct functions should pass the address as payable:

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L110

selfdestruct(payable(_sale.saleReceiver));

