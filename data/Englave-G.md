#### Functions that can be declared external
`public` functions never called by the contract should be declared `external` to save gas.
Solidity immediately copies array arguments to memory in public functions, while external functions can read directly from the calldata. Memory allocation is expensive, whereas reading from calldata is cheap.

**Path:** ./minters/FixedPrice.sol : initialize(), getPrice(), startTime(), editionContract(), available();
./minters/FixedPriceFactory.sol : setFeeReceiver();
./Escher721.sol : mint(), updateURIDelegate(), setDefaultRoyalty(), resetDefaultRoyalty();
./Escher.sol : setURI(), addCreator();
./minters/OpenEditionFactory.sol : setFeeReceiver();
./minters/OpenEdition.sol: initialize(), finalize(), getPrice(), startTime(), timeLeft(), editionContract(), available();
./minters/LPDAFactory.sol : setFeeReceiver();
./minters/LPDA.sol : refund(), initialize(), getPrice(), startTime(), editionContract(), available(), lowestPrice();
**Recommendation:** Use external instead of public