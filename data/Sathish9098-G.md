
##

## [GAS-1]   FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

> instances (2) 

[File: 2022-12-escher/src/uris/Generative.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/uris/Generative.sol)

        14:   function setScriptPiece(uint256 _id, string memory _data) external onlyOwner {

        19:   function setSeedBase() external onlyOwner {

 ##

##  [GAS-2]  Use the external Visibility Modifier

Use the external function visibility for gas optimization because the public visibility modifier is equivalent to using the external and internal visibility modifier, meaning both public and external can be called from outside of your contract, which requires more gas.

  Remember that of these two visibility modifiers, only the public modifier can be called from other functions inside of your contract.  

> instances (2)      

[File: 2022-12-escher/src/minters/LPDAFactory.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol)

         46:     function setFeeReceiver(address payable fees) public onlyOwner {

[File: 2022-12-escher/src/Escher.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol)

        27:        function addCreator(address _account) public onlyRole(CURATOR_ROLE) {

##

## [GAS-3]   Compute known value off-chain

If You know what data to hash, there is no need to consume more computational power to hash it using keccak256 , you’ll end up consuming 2x amount of gas.

> instances (4) 

[File: 2022-12-escher/src/Escher.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol)

     11:  bytes32 public constant CREATOR_ROLE = keccak256("CREATOR_ROLE");

    13:   bytes32 public constant CURATOR_ROLE = keccak256("CURATOR_ROLE");

[File: 2022-12-escher/src/Escher721.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721.sol)

    17:    bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");
    
    19:    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

##

## [ GAS-4]  ++I/I++ OR --I/I-- SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} OR  UNCHECKED{--I}/UNCHECKED{I--}WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

> instances (3) 

[File: 2022-12-escher/src/minters/FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)

          65:    for (uint48 x = sale_.currentId + 1; x <= newId; x++) {

[File: 2022-12-escher/src/minters/OpenEdition.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol)

        66:     for (uint24 x = temp.currentId + 1; x <= newId; x++) {

[File: 2022-12-escher/src/minters/LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

       73:     for (uint256 x = temp.currentId + 1; x <= newId; x++) {

##

## [GAS - 7]  SHOULD USE LOCAL VARIBALE INSTEAD OF STATE VARIABLE . 

This will save near 97 gas

    [File: 2022-12-escher/src/minters/FixedPrice.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol)

          71:   emit Buy(msg.sender, _amount, msg.value, sale);

##

##  [GAS-8]   <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES . FOR EVERY CALL CAN SAVE 13 GAS 

> instances (3) 

 [File: 2022-12-escher/src/minters/LPDA.sol](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol)

         66:   amountSold += amount;
        
         70:   receipts[msg.sender].amount += amount;

         71:    receipts[msg.sender].balance += uint80(msg.value);



        

 


           





































