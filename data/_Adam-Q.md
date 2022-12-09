### [L01] Use safeMint instead of mint for ERC721
**Description:**
If a user tries to buy one of the nft editions from a contract address that does not support ERC721 tokens, the NFT can be frozen in the contract.

**LOC:**
[FixedPrice.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L66)
[LPDA.sol#L74](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L74) 
[OpenEdition.sol#L67](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L67) 

**Recommendation:**
Change mint to safeMint.


### [L02] Selfdestruct is deprecated
**Description:**
As per [EIP-6049](https://eips.ethereum.org/EIPS/eip-6049) & [EIP-4758](https://eips.ethereum.org/EIPS/eip-4758) selfdestruct is being deprecated and it is recommended not to use. 

**LOC:**
[FixedPrice.sol#L110](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L110) 
[OpenEdition.sol#L122](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L122) 

**Recommendation:**
Instead of using selfdestruct to send the funds to the saleReciever replace the lines with: (bool success, ) = saleReciever.call{value: address(this).balance}("");
	  require(success, "txn failed"); 


### [L03] Unlocked Pragma
**Description:**
Non Library/Interface contracts should be deployed with a locked pragma version. This prevents the contract being deployed with a version that wasn't thoroughly tested against in development.

**LOC:**
[Escher.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L2) 
[Escher721.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L2) 
[Escher721Factory.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L2) 
[FixedPrice.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L2) 
[FixedPriceFactory.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L2) 
[LPDA.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L2) 
[LPDAFactory.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L2)
[OpenEdition.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L2) 
[OpenEditionFactory.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L2) 
[Base.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Base.sol#L2)
[Generative.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L2) 
[Unique.sol#L2](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Unique.sol#L2) 

**Recommendation:**
Lock the pragma to the version that was used in testing.

### [L04] Use call over transfer
**Note:** This issue is mentioned in the auto generated report but it is incorrectly labeled as a unsafe ERC20 transfer, so i have included it in my QA report. If i wasn't supposed to I apologise and please ignore.

**Description:**
The transfer function as a gas limit of 2,300 which can be an issue when sending ETH to smart contract addresses. This can be a result of the recieving smart contract not implementing a payable fallback function or the cumulative gas costs of the transaction exceeding the 2,300 limit.

**LOC:**
[FixedPrice.sol#L109](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L109) 
[LPDA.sol#L85-L86](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L85-L86) 
[LPDA.sol#L105](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L105) 
[OpenEdition.sol#L92](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L92) 

**Recommendation:**
Change transfer to call.