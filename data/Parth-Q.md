* Missing zero check on `_sale.saleReceiver` and `_sale.price`. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L29-L38 

* Missing zero check for `sale.finalPrice` https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L29-L42

* Missing zero check on `sale.saleReceiver` and `sale.price` https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L29-L38