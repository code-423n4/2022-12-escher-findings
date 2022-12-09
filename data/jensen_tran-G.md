#1 Loop Optimization
Issue: 
for (uint256 x = temp.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
}
Should be:
- cache array length = newId + 1 then using x < length.
- increase x in unchecked block.
Solution:
uint256 length = newId + 1;
for (uint256 x = temp.currentId + 1; x <length;) {
            nft.mint(msg.sender, x);
            unchecked {
                 ++x;
            }
}
#2 Setup Optimization
Issue:
sale = Sale(
            type(uint72).max,
            type(uint24).max,
            address(0),
            type(uint96).max,
            payable(0),
            type(uint96).max
        );
Should be: 
- Avoid init values with zero values, cause later initialize would store new value which is non-zero. Convert from zero value to non-zero value cost more gas.
Solution:
sale = Sale(
            type(uint72).max,
            type(uint24).max,
            0x06058942EF19dA2054Fb669793D3889dE42d6c38, // anything but not zero
            type(uint96).max,
            payable(0x06058942EF19dA2054Fb669793D3889dE42d6c38), // anything but not zero
            type(uint96).max
        );
#3 Using memory variable instead of storage
OpenEdition.sol -> Line 63
require(amount * sale.price == msg.value, "WRONG PRICE");
Should be:
- Using temp variable which is cache of sale.
Solution:
require(amount * temp.price == msg.value, "WRONG PRICE");


