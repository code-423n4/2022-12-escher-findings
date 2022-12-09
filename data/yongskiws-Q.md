Q&N
## Consider Functions that send Ether to arbitrary destinations (arbitrary-send-eth)
Fixedprice.sol,Open Edition.sol
```solidity
    function _end(Sale memory _sale) internal {
        emit End(_sale);
        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
        selfdestruct(_sale.saleReceiver);
    }
```
    function finalize() public {
        Sale memory temp = sale;
        require(block.timestamp >= temp.endTime, "TOO SOON");
        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
        _end(temp);
    }

## Weak PRNG (weak-prng)
generative.sol

    function setSeedBase() external onlyOwner {
        require(seedBase == bytes32(0), "SEED BASE SET");

        uint256 time = block.timestamp;
        uint256 numb = block.number;
        seedBase = keccak256(abi.encodePacked(numb, blockhash(numb - 1), time, (time % 200) + 1));
    }

## Dangerous usage of `block.timestamp` (timestamp)

fixedprice.sol
require(block.timestamp >= sale_.startTime, "TOO SOON");
require(block.timestamp < sale.startTime, "TOO LATE");
require(block.timestamp < sale.startTime, "TOO LATE");

fixedpricefactory.sol
require(_sale.startTime >= block.timestamp, "START TIME IN PAST");

## Multiple calls in a loop (calls-loop)
fixedprice.sol
 for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }

## Uses nonreentrant or guard
fixedprice.sol
    function buy(uint256 _amount) external payable {