### 01 _SAFEMINT() SHOULD BE USED RATHER THAN _MINT() WHEREVER POSSIBLE

*Number of Instances Identified: 3*

`_mint()` is discouraged in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both open OpenZeppelin and solmate have versions of this function so that NFTs aren't lost if they're minted to contracts that cannot transfer them back out.

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol

```
63:  _mint(_account, uint256(_role), 1, "0x0");
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol

```
66: nft.mint(msg.sender, x);
```


https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol

```
51-52: function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {
        _mint(to, tokenId);
```



### 02 MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS

*Number of Instances Identified: 8*

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol

```
64: function setDefaultRoyalty(
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol

```
42: function setFeeReceiver(address payable fees) public onlyOwner {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol

```
46: function setFeeReceiver(address payable fees) public onlyOwner {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol

```
21: function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol

```
42: function setFeeReceiver(address payable fees) public onlyOwner {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol

```
10: function setBaseURI(string memory _baseURI) external onlyOwner {
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol

```
14: function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {
19: function setSeedBase() external onlyOwner {
```



### 03 MISSING CHECKS FOR ADDRESS(0X0) WHEN ASSIGNING VALUES TO ADDRESS STATE VARIABLES

*Number of Instances Identified: 4*

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol

```
57-58: function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {
        tokenUriDelegate = _uriDelegate;
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol

```
42-43: function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol

```
46-47: function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol

```
42-43: function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
```


### 04 NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

### Proof of Concept

This issue exists in the following In-scope contracts :

[FixedPriceFactory.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol)), [LPDA.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)), [Escher.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol)), [Escher721.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol)), [Escher721Factory.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol)), [FixedPrice.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)),  [LPDAFactory.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)), [OpenEdition.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)), [OpenEditionFactory.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol))
[Unique.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Unique.sol)) [Base.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Base.sol)), [Generative.sol]([https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol))


### Recommended Mitigation Steps

Lock the pragma version


