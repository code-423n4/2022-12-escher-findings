## [L-01] ERC721 _safeMint should be used instead of _mint whenever possible

_mint() is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both open OpenZeppelin and solmate have versions of this function so that NFTs aren't lost if they're minted to contracts that cannot transfer them back out.

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L51-L53
```
    function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) {
        _mint(to, tokenId);
    }
```