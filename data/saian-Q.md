
# Low Severity findings

## Use safeMint instead of mint

In Escher721.sol function mint is used to mint tokens to the receiver. If this address is a contract that does not handle ERC721 tokens, the minted tokens will be stuck in the contract and cannot be retrieved.

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol#L52

```
    _mint(to, tokenId);
```

### Recommended Mitigation

Replace `mint` with `safeMint`

## LPDA.sol `buy` and `refund` will revert with high dropPerSecond

In LPDA.sol if dropPerSecond is set to a high value, functions `getPrice` and `refund` and  will revert if `startPrice` becomes less than `dropPerSecond*time`. The sale cannot be ended and users will not be able to refund the excess ether sent during buy, ether will be stuck in the contract.

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L124

```
        return temp.startPrice - (temp.dropPerSecond * timeElapsed);
```

### Recommended Mitigation

Add condition to validate dropPerSecond in LPDAFactory

```
    require(sale.startPrice > (sale.dropPerSecond * (sale.endTime - sale.startTime)));

```
## Replace `transfer()` with `call()`

Since transfer forwards a fixed amount of gas 2300, the calls may fail if gas costs change in the future, or if the receiver is a smart contract that consumes more than 2300 gas.
Use `.call()` to send ether instead of transfer.

refer-https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L109

```
        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L92

```
        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L85-L86

```
        ISaleFactory(factory).feeReceiver().transfer(fee);
        temp.saleReceiver.transfer(totalSale - fee);
```

### Recommended Mitigation

Replace `transfer` with `call`

## Add zero address checks to saleReceiver address

In `FixedPrice` and `OpenEdition` contracts ownership is transferred to saleReceiver during intialization. If the saleReceiver address is accidentally set to address(0), user will be unable to cancel the sale, and ether from the sale will be sent to address(0)

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L50

```
    function cancel() external onlyOwner {
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L30-L32

```
    require(IEscher721(_sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
    require(_sale.startTime >= block.timestamp, "START TIME IN PAST");
    require(_sale.finalId > _sale.currentId, "FINAL ID BEFORE CURRENT");
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L29

```
    require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
    require(sale.startTime >= block.timestamp, "START TIME IN PAST");
    require(sale.endTime > sale.startTime, "END TIME BEFORE START");
```

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L31

```
    require(sale.saleReceiver != address(0), "INVALID SALE RECEIVER");
```
### Recommended Mitigation

Add condition to validate saleReceiver address

```
    require(sale.saleReceiver != address(0), "INVALID SALE RECEIVER");
```