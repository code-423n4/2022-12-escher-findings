# QA Report

1. Non-library/interface files should use fixed compiler versions, not floating ones
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol#L2
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol#L2
2. Events missing indexed fields
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L31
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L34
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L41
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L42
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L30
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L33
3. Function order is incorrect
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L14-L24
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L13-L27
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L13-L23
4. Missing zero address checks
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L29
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L29
   - https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L29
