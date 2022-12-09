# Low/QA report by stealthyz

## QA/Low Escher721.sol '_burn()' function not used 
Line 96:
```
function _burn(uint256 tokenId) internal override(ERC721Upgradeable) {
    super._burn(tokenId);
}
```

This function is not used and can be removed. It only calls the inhertied function which makes it useless. 

### Impact

Makes the deployment cost larger and inflates the bytecode. This function doesn't check if the sender has rights to operate on provided token ID. 
If this will be implemented then it is required to check the aforementioned privilege.

------

## QA/Low: Escher.sol '_revokeRole()' function not used

Escher.sol line 67:
```
function _revokeRole(bytes32 _role, address _account) internal override {
    super._revokeRole(_role, _account);
    _burn(_account, uint256(_role), balanceOf(_account, uint256(_role)));
}
```
This function is not used and thus can be removed. It doesn't check if '_burn' is called by someone other thatn 'account' and thus could be a security
risk if not correctly implemented. 

------------
