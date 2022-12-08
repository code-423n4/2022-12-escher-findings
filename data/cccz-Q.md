## [Low-01]  PRNG in Generative.setSeedBase

### Impact
Generative.setSeedBase uses block.timestamp/block.number/blockhash to generate random numbers, which results in the generated random numbers being predictable, and since the generated random numbers are generally used to determine the properties of NFTs, users can predict the random numbers and thus purchase rarer NFTs.

### Proof of Concept
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L19-L25

### Recommended Mitigation Steps
Consider using chainlink VRF for random number generation
https://docs.chain.link/vrf/v2/introduction/


## [Low-02] _safeMint() should be used rather than _mint() wherever possible

### Impact

_mint() is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver.

### Proof of Concept
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L66-L67
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L74-L75
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L67-L68

### Recommended Mitigation Steps

Use _safeMint instead of _mint


## [Low-03] LPDAFactory.createLPDASale should check startPrice >= dropPerSecond * (end-start)

### Impact
LPDAFactory.createLPDASale should check startPrice >= dropPerSecond * (end-start), if startPrice < dropPerSecond * (end-start), then LPDA.getPrice() may revert due to overflow, and buy()/refund() will fail, making ETH locked in the contract
```solidity
   function getPrice() public view returns (uint256) {
        Sale memory temp = sale;
        (uint256 start, uint256 end) = (temp.startTime, temp.endTime);
        if (block.timestamp < start) return type(uint256).max;
        if (temp.currentId == temp.finalId) return temp.finalPrice;

        uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;
        return temp.startPrice - (temp.dropPerSecond * timeElapsed);
    }
```

### Proof of Concept
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L117-L125

### Recommended Mitigation Steps
Consider checking startPrice >= dropPerSecond * (end-start) in LPDAFactory.createLPDASale




## [Low-04] Unable to create sale for NFT with id 0

### Impact
When creating a sale, the currentId is at least 0, but in the buy function, only the NFT with ID currentId + 1 will be minted, which makes it impossible to create a sale for an NFT with ID 0
```solidity
    function buy(uint256 _amount) external payable {
        Sale memory sale_ = sale;
        IEscher721 nft = IEscher721(sale_.edition);
        require(block.timestamp >= sale_.startTime, "TOO SOON");
        require(_amount * sale_.price == msg.value, "WRONG PRICE");
        uint48 newId = uint48(_amount) + sale_.currentId;
        require(newId <= sale_.finalId, "TOO MANY");

        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
            nft.mint(msg.sender, x);
        }
```

### Proof of Concept
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L57-L67

### Recommended Mitigation Steps
Consider allowing NFTs with ID 0 to be minted in the buy function