
## 01 _SAFEMINT() SHOULD BE USED RATHER THAN _MINT() WHEREVER POSSIBLE

`_mint()` is discouraged in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both open OpenZeppelin and solmate have versions of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.

_There are 3 instances of this issue_

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol

```
File: src/Escher.sol

63:  _mint(_account, uint256(_role), 1, "0x0");
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol

```
File: src/Escher721.sol

51-52: function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {
        _mint(to, tokenId);
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol

```
File: src/minters/FixedPrice.sol

66: nft.mint(msg.sender, x);
```

-----------

## 02 MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS

The afunctions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

Missing events and timelocks do not promote transparency and if such changes immediately affect users’ perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.

_There are 8 instances of this issue_

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol

```
File: src/Escher.sol

21: function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol

```
File: src/Escher721.sol

64: function setDefaultRoyalty(
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol

```
File: src/minters/FixedPriceFactory.sol

42: function setFeeReceiver(address payable fees) public onlyOwner {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol

```
File: src/minters/LPDAFactory.sol

46: function setFeeReceiver(address payable fees) public onlyOwner {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol

```
File: src/minters/OpenEditionFactory.sol

42: function setFeeReceiver(address payable fees) public onlyOwner {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol

```
File: src/uris/Base.sol

10: function setBaseURI(string memory _baseURI) external onlyOwner {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol

```
File: src/uris/Generative.sol

14: function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {
19: function setSeedBase() external onlyOwner {
```

-------------

## 03 TRANSFEROWNERSHIP SHOULD BE TWO STEP

The owner is the authorized user in the solidity contracts. Usually, an owner can be updated with transferOwnership function. However, the process is only completed with single transaction. If the address is updated incorrectly, an owner functionality will be lost forever.

_There are 4 instances of this issue_

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol

```
File: src/minters/FixedPrice.sol

80: _transferOwnership(_sale.saleReceiver);
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol

```
File: src/minters/LPDA.sol

112:  _transferOwnership(sale.saleReceiver);
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol

```
File: src/minters/OpenEdition.sol

84: _transferOwnership(sale.saleReceiver);
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol

```
File: uris/Base.sol

19: _transferOwnership(_owner);
```

### Recommended Mitigation Steps

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

---------

## 04 NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

### Proof of Concept

This issue exists in the following In-scope contracts :

[Unique.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol), [Base.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol), [Generative.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol), [Escher.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol), [Escher721.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol), [Escher721Factory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol), [FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol), [FixedPriceFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol), [LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol), [LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol), [OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol), [OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol)

### Recommended Mitigation Steps

Lock the pragma version

--------

## 05 MISSING CHECKS FOR ADDRESS(0X0) WHEN ASSIGNING VALUES TO ADDRESS STATE VARIABLES

_There are 4 instances of this issue_

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol

```
File: src/Escher721.sol

57-58: function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {
        tokenUriDelegate = _uriDelegate;
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol

```
File: src/minters/FixedPriceFactory.sol

42-43: function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol

```
File: src/minters/LPDAFactory.sol

46-47: function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol

```
File: src/minters/OpenEditionFactory.sol

42-43: function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
```

------
