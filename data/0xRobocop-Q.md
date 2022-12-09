# Escher.sol:

## 1.- uri is the same for the creator token and curator token.

At [LoC 21](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L21) the `function setURI(string memory _newuri)` is used to set the uri of the collection. This function will set the same uri for all the tokens. The openzeppelin documentation clearly says that is the responsibility of the client fetching the uri to replace the '/id/' part of the uri. The documentation of Escher does not say if this is the behavior they want, or they prefer a different uri for each token id in case they use arweave for example. 

## 2.- curators can grant themselves the creator role.

At [LoC 27](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L27) the `function addCreator(address _account)` allows only to the curators to grant an account the role of creator. In the current implementation, a curator can give himself the role of creator:

```
function addCreator(address _account) public onlyRole(CURATOR_ROLE) {
        _grantRole(CREATOR_ROLE, _account);
    }
```
 Escher lacks documentation if this is the behavior they want or not.

# Escher721.sol:

## 3.- Creator of an Escher721 edition cannot set royalty information for tokens individually.

The documentation says that a creator can call the function `setTokenRoyalty()` with three parameters: id, receiver, feeNumerator. This is to set the royalty information for the given id. But the Escher721.sol only declares an external function for `setDefaultRoyalty()` which sets the same royalty information for all the tokens. The ERC2981 contract inherited from openzeppelin only contains an internal version of `setTokenRoyalty()`, so it needs to declare an external function which invokes the internal one.