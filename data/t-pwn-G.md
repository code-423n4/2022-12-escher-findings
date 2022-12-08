++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)
Saves 5 gas per loop

File: src/minters/FixedPrice.sol

65:         for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
File: src/minters/LPDA.sol

73:         for (uint256 x = temp.currentId + 1; x <= newId; x++) {
File: src/minters/OpenEdition.sol

66:         for (uint24 x = temp.currentId + 1; x <= newId; x++) {