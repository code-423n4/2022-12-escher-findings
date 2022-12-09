### [L-01] Avoid floating pragmas for non-library contracts.


#### Impact
While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### Findings:
```
src/Escher.sol:L2     pragma solidity ^0.8.17;

src/Escher721.sol:L2     pragma solidity ^0.8.17;

src/Escher721Factory.sol:L2     pragma solidity ^0.8.17;

src/minters/FixedPriceFactory.sol:L2     pragma solidity ^0.8.17;

src/minters/OpenEdition.sol:L2     pragma solidity ^0.8.17;

src/minters/LPDAFactory.sol:L2     pragma solidity ^0.8.17;

src/minters/FixedPrice.sol:L2     pragma solidity ^0.8.17;

src/minters/LPDA.sol:L2     pragma solidity ^0.8.17;

src/minters/OpenEditionFactory.sol:L2     pragma solidity ^0.8.17;

src/uris/Unique.sol:L2     pragma solidity ^0.8.17;

src/uris/Base.sol:L2     pragma solidity ^0.8.17;

src/uris/Generative.sol:L2     pragma solidity ^0.8.17;

```

### [L-02] ```_safemint()``` should be used rather than ```_mint()``` wherever possible


#### Impact
```_mint()``` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of ```_safeMint()``` which ensures that the recipient is either an EOA or implements ```IERC721Receiver```. Both [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and [solmate](https://github.com/transmissions11/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function


#### Findings:
```
src/Escher.sol:L63        _mint(_account, uint256(_role), 1, "0x0");

src/Escher721.sol:L52        _mint(to, tokenId);

```
