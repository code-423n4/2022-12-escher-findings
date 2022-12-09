1:
Sale contracts are at risk of not being completed

FixedPrice.sol/LPDA.sol The condition for the owner to get back the sale funds is: finalId==currentId
OpenEdition.sol: The owner gets back the sale funds is:  block.timestamp >= sale.endTime

But for both types of contracts, there is no restriction that the user's findalId can be set very large, or the endTime can be set very large, so that the sale may never be completed.
For example:
currentId = 0 , finalId = type(uint256).max
endTime = type(uint96).max

These are the cases that may result in not being able to complete the sale. Then the sold funds will be locked in the contract.

It is recommended that FixedPrice.sol/LPDA.sol/OpenEdition.sol add the "maximum end time" to one year, so that after one year , the owner can choose whether to end the sale and get back the sold funds


2:
OpenEdition#available() calculation error, not subtracted currentId
```solidity
contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
...
    function available() public view returns (uint256) {
- return sale.endTime > block.timestamp ? type(uint24).max : 0; 
+ return sale.endTime > block.timestamp ? type(uint24).max - sale.currentId : 0;
    }
```

3:
OpenEditionFactory/LPDAFactory/FixedPriceFactory
Detect that receive cannot be address(0)
```solidity
    function createOpenEdition(OpenEdition.Sale calldata sale) external returns (address clone) {
        require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
        require(sale.startTime >= block.timestamp, "START TIME IN PAST");
        require(sale.endTime > sale.startTime, "END TIME BEFORE START");

+       require(sale.saleReceiver ! = address(0), "INVALID SALE RECEIVER");