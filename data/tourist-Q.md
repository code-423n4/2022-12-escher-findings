## [N-01] WRONG FUNCTION COMMENTS

```solidity
File:  src/minters/LPDA.sol

34    /// @notice sale info for this fixed price edition
56    /// @notice buy from a fixed price sale after the sale starts
91    /// @notice cancel a fixed price sale
```

```solidity
File:  src/minters/LPDAFactory.sol

27    /// @notice create a fixed sale proxy contract
44   /// @notice set the fee receiver for fixed price editions
```

```solidity
File:  src/minters/OpenEdition.sol

25    /// @notice sale info for this fixed price edition
55    /// @notice buy from a fixed price sale after the sale starts
74    /// @notice cancel a fixed price sale
```


```solidity
File:  src/minters/OpenEditionFactory.sol

27    /// @notice create a fixed sale proxy contract
40   /// @notice set the fee receiver for fixed price editions
```

Comments are copied from ```FixedPrice.sol``` and ```FixedPriceFactory.sol``` contracts and should be changed according to the new contract.

## [N-02]  WRONG FIRST NOTICE FOR ```getPrice()``` function in ```FixedPrice.sol``` contract

```solidity
File: src/minters/FixedPrice.sol

84    /// @notice cancel a fixed price sale
85    /// @notice the price of the sale
```

The first notice comment should be removed.