### Summary

Overall the codebase is quite readable, well organized. For the most part the codebase closley matched the documenation but there where a few mismatches that were mentioned below. Project structure is intuitive and simple to follow. Contract structure is well ogranized and makes good use of libraries and external contracts. Variable naming is mostly easily readble and intuitive. Comments were used rigourously and were beneficial and accurate in describing the object they were comenting on. Events are used in a benefical manner to track protocol outputs.
   
   

### Low Quality issues
#### [LQ-01] No external/public function for {_setTokenRoyalty}

No external/public function for {_setTokenRoyalty} as mentioned in the sale flow of the docs. Currently royalty can only be set through default and all token ids.

instances:1

File: Escher721.sol
Line: N/A
[Link](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721.sol)


#### [LQ-02] Improved flow to grant the proxy minter role
Better flow to grant the proxy minter role (escher721 minting function) during initialize instead of having to call it separately. As either way it will have to be called in order for the buy function to work

instances:3

File: FixedPrice.sol
Line: 78-82
[Link](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L78-L82)

```javascript
    function initialize(Sale memory _sale) public initializer {
        sale = _sale;
        _transferOwnership(_sale.saleReceiver);
        emit Start(_sale);
    }
```

File: LPDA.sol
Line: 110-114
[Link](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L110-L114)

```javascript
    function initialize(Sale memory _sale) public initializer {
        sale = _sale;
        _transferOwnership(_sale.saleReceiver);
        emit Start(_sale);
    }
```

File: OpenEdition.sol
Line: 82-86
[Link](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L82-L86)

```javascript
    function initialize(Sale memory _sale) public initializer {
        sale = _sale;
        _transferOwnership(_sale.saleReceiver);
        emit Start(_sale);
    }
```

### Non-Critical issues
#### [NC-01]

For better readability, adding comment that curators will be added by admins manually calling AccessControl "grantRole" function.

Comment could be added above the addCreator function.

instances:1

File: Escher.sol
Line: 27
[Link](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L27-L29)


#### [NC-02]

Typo in the comment. comment mentions "set the prices" i believe it should be "set the pieces"

instances:1

File: Generative.sol
Line: 11

#### [NC-03]

For better readability, fees parameter could be named feesAdress as fees implies an amount

instances:3

File: FixedPriceFactory.sol
Line: 40-44
[Link](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L40-L44)

```javascript
    /// @notice set the fee receiver for fixed price editions
    /// @param fees the address to receive fees
    function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
    }
```

File: LPDAFactory.sol
Line: 44-47
[Link](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L44-L47)

```javascript
    /// @notice set the fee receiver for fixed price editions
    /// @param fees the address to receive fees
    function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
    }
```

File: OpenEditionFactory.sol
Line: 40-44
[Link](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L40-L44)

```javascript
    /// @notice set the fee receiver for fixed price editions
    /// @param fees the address to receive fees
    function setFeeReceiver(address payable fees) public onlyOwner {
        feeReceiver = fees;
    }
```

#### [NC-04]

Fixed price sale in the comment should be Open Edition sale

instances:2

File: FixedPriceFactory.sol
Line: 27 && 40


#### [NC-05]

Missing check to on Zero address for _sale.saleReceiver. Mostly for a precaution as a zero address can lead to situation where sale funds are lost.

instances:2

File: OpenEditionFactory.sol
Line: 29-38
[Link](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L29-L38)

```javascript
    function createOpenEdition(OpenEdition.Sale calldata sale) external returns (address clone) {
        require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
        require(sale.startTime >= block.timestamp, "START TIME IN PAST");
        require(sale.endTime > sale.startTime, "END TIME BEFORE START");


        clone = implementation.clone();
        OpenEdition(clone).initialize(sale);


        emit NewOpenEditionContract(msg.sender, sale.edition, clone, sale);
    }
```

File: FixedPriceFactory.sol
Line: 29-38
[Link](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L29-L38)

```javascript
    function createOpenEdition(OpenEdition.Sale calldata sale) external returns (address clone) {
        require(IEscher721(sale.edition).hasRole(bytes32(0x00), msg.sender), "NOT AUTHORIZED");
        require(sale.startTime >= block.timestamp, "START TIME IN PAST");
        require(sale.endTime > sale.startTime, "END TIME BEFORE START");


        clone = implementation.clone();
        OpenEdition(clone).initialize(sale);


        emit NewOpenEditionContract(msg.sender, sale.edition, clone, sale);
    }
```