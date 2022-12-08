### 1st Bug
Unsafe ERC20 Operation(s)

#### Impact
Issue Information: 
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.

Example
🤦 Bad:

IERC20(token).transferFrom(msg.sender, address(this), amount);
🚀 Good (using OpenZeppelin's SafeERC20):

import {SafeERC20} from "openzeppelin/token/utils/SafeERC20.sol";

// ...

IERC20(token).safeTransferFrom(msg.sender, address(this), amount);
🚀 Good (using require):

bool success = IERC20(token).transferFrom(msg.sender, address(this), amount);
require(success, "ERC20 transfer failed");

#### Findings:
```
2022-12-escher/src/minters/FixedPrice.sol::109 => ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
2022-12-escher/src/minters/LPDA.sol::85 => ISaleFactory(factory).feeReceiver().transfer(fee);
2022-12-escher/src/minters/LPDA.sol::86 => temp.saleReceiver.transfer(totalSale - fee);
2022-12-escher/src/minters/LPDA.sol::105 => payable(msg.sender).transfer(owed);
2022-12-escher/src/minters/OpenEdition.sol::92 => ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```
#### Tools used
Manual




### 2nd Bug
Unspecific Compiler Version Pragma

#### Impact
Issue Information: 
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

Example
🤦 Bad:

pragma solidity ^0.8.0;
🚀 Good:

pragma solidity 0.8.4;
#### Findings:
```
2022-12-escher/src/Escher.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/Escher721.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/Escher721Factory.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/FixedPrice.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/FixedPriceFactory.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/LPDA.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/LPDAFactory.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/OpenEdition.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/minters/OpenEditionFactory.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/uris/Base.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/uris/Generative.sol::2 => pragma solidity ^0.8.17;
2022-12-escher/src/uris/Unique.sol::2 => pragma solidity ^0.8.17;
```
#### Tools used
Manual