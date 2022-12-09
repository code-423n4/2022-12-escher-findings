## 1. ++I COSTS LESS GAS COMPARED TO I++ (SAME FOR --I VS I-- ) and INCREMENTS/DECREMENTS CAN BE UNCHECKED IN FOR-LOOPS

        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {

This can be changed to 

        for (uint48 x = sale_.currentId + 1; x <= newId;) {
             unchecked{ ++x; }
        }

**There are three instances of this issue:**

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66

## 2. > 0 IS LESS EFFICIENT THAN != 0 FOR UNSIGNED INTEGERS

        require(owed > 0, "NOTHING TO REFUND");

This can be changed as follows:

        require(owed != 0, "NOTHING TO REFUND");

**There are three instances of this issue:**

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L103

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L35

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L36
