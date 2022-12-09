# Total Gas Savings

The below changes led to:

Deployment Cost Reduction
"Runtime" Reduction
XXX Reduction 

# Gas Optimizations

## Increment in For Loop Can Go Unchecked

For Solidity >8.0, arithmetic is automatically checked for underflows and overflows.  However, this costs marginally more amount of gas.  Using the keyword, unchecked, will forego this check.  Since the increment in the for loop will never overflow, it can go unchecked, saving gas. 

    @@ -62,8 +62,9 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
 
    -        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
    +        for (uint48 x = sale_.currentId + 1; x <= newId;) {
                 nft.mint(msg.sender, x);
    +            unchecked{++x;}
         }

Number of Instances: 3

Lines of Code: 

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65-L67
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73-L75
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66-L68


## uint256 initialized to 0

The default value for a uint256 is zero.  Setting it to 0 will use unnecessary STORE and MSTORE opcodes, which can cost a minimum of 100 and 3 gas, respectively.  

    @@ -33,7 +33,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
 
         /// @notice sale info for this fixed price edition
         Sale public sale;
    -    uint48 public amountSold = 0;
    +    uint48 public amountSold;

Number of Instances: 1

Lines of Code: 

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L36

## ++I Costs Less Gas Than I++ in For Loops

Using ++I rather than I++ saves 5 gas for each iteration of a for loop

      @@ -62,7 +62,7 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
             uint48 newId = uint48(_amount) + sale_.currentId;
             require(newId <= sale_.finalId, "TOO MANY");
 
    -        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
    +        for (uint48 x = sale_.currentId + 1; x <= newId; ++x) {
                 nft.mint(msg.sender, x);
             }
 
Number of Instances: 3

Lines of Code:

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65-L67
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73-L75
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66-L68

## Use for(uint256 x; x < y; x++) in place of for(uint256 x + 1; x <= y; x++)

Using for(uint256x; x < y; x++) is more gas efficient than for(uint256 x + 1; x <= y; x++).  Specifically, > will be used in place of >= and an unnecessary addition will be omitted.  This will save a minimum of 3 gas for each iteration of the loop and 3 gas for each function call.  

However, this changes the aforementioned contracts' behavior, since token ids can now begin at zero if the creator sets a sale's currentId to 0.  The state variable, currentId, should be renamed to nextId to reflect this behavior.

    @@ -13,7 +13,7 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
         /// avoids strict equality of current == final
         struct Sale {
             // slot 1
    -        uint48 currentId;
    +        uint48 nextId;
         
     @@ -59,14 +59,14 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
             IEscher721 nft = IEscher721(sale_.edition);
             require(block.timestamp >= sale_.startTime, "TOO SOON");
             require(_amount * sale_.price == msg.value, "WRONG PRICE");
    -        uint48 newId = uint48(_amount) + sale_.currentId;
    +        uint48 newId = uint48(_amount) + sale_.nextId;
             require(newId <= sale_.finalId, "TOO MANY");
 
    -        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
    +        for (uint48 x = sale_.nextId; x < newId; x++) {
                 nft.mint(msg.sender, x);
             }
 
    -        sale.currentId = newId;
    +        sale.nextId = newId;

Instances: 3 

Lines of Code:

** Add lines of code later

## Inefficient Variable Packing

The last slot in Sale structs use uint96 rather than uint256, even though they both use a full storage slot.  Since the EVM operates in 32 bytes at a type, using a uint96 over a uint256 will lead to unnecessary conversion costs.  

    @@ -20,7 +20,7 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
              struct Sale {
            // slot 1
            uint48 currentId;
            uint48 finalId;
            address edition;
            // slot 2
             uint96 price;
             address payable saleReceiver;
             // slot 3
    -        uint96 startTime;
    +        uint256 startTime;
         }

Instances: 

Lines of Code:

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L23
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L26
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L22

## Unnecessary SLOADs and MSTOREs

buy( ) in LPDA has an unnecessary SLOADs and MSTOREs.  Specifically, getPrice( ) loads the state variable, sale.  However, this is already loaded at the beginning of buy( ).  The solution is to split getPrice( ) into two functions: getPrice( ) and _getPrice( ) so that the latter uses a memory struct.

      @@ -60,7 +60,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
             Sale memory temp = sale;
             IEscher721 nft = IEscher721(temp.edition);
             require(block.timestamp >= temp.startTime, "TOO SOON");
    -        uint256 price = getPrice();
    +        uint256 price = _getPrice(temp);
              require(msg.value >= amount * price, "WRONG PRICE");
 
             amountSold += amount;
     @@ -115,7 +115,10 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
 
         function getPrice() public view returns (uint256) {
     -        Sale memory temp = sale;
    +        return _getPrice(sale);
    +    }
    +    
    +    function _getPrice(Sale memory temp) internal view returns(uint256) {
             (uint256 start, uint256 end) = (temp.startTime, temp.endTime);
             if (block.timestamp < start) return type(uint256).max;
             if (temp.currentId == temp.finalId) return temp.finalPrice; 

            uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;
            return temp.startPrice - (temp.dropPerSecond * timeElapsed);
        }

Number of Instances: 1

Lines of Code: 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L117-L126

## Use x = x + y rather than x += y for state variables

x += y costs more than x = x + y for state variables.

Instances: 3

Referenced Code: 
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L66-L71

## Custom Errors 

Custom errors in Solidity are much more gas efficient in terms of storage than using strings.  While using strings for errors may make identifying them easier, custom errors are relatively easy to calculate: they are simply the function selector of the error.  

    @@ -6,6 +6,7 @@ import {Escher721} from "./Escher721.sol";
     import {Clones} from "openzeppelin/proxy/Clones.sol";
     import {ITokenUriDelegate} from "./interfaces/ITokenUriDelegate.sol";
 
    +error NotAuthorized();
 
    @@ -29,7 +30,7 @@ contract Escher721Factory {
    -        require(escher.hasRole(escher.CREATOR_ROLE(), msg.sender), "NOT AUTHORIZED");
    +        if(!(escher.hasRole(escher.CREATOR_ROLE(), msg.sender))) revert NotAuthorized();

Number of Instances: 32

Lines of Code: 

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L32
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher.sol#L61
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L15
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L20
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L28
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L51
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L60-L63
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L30-L32
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L62-L68
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L93
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L103
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L30-L36
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L76
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L61-L63
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L91
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L30-L32


# Full Git Diff

The full git diff of my gas optimized code can be found here:

