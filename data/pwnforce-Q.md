1- Zero-address checks as input validation on address parameters is always a best practice. This is especially true for critical addresses that are immutable and set in the constructor because they cannot be changed later. Indeed, constructors should check the address written in an immutable address variable is not the zero address:
_escher in line 18, Escher721Factory.sol needs to have a zero address check. 

2- Using transfer() function is no longer recommended for sending Ether (https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/):

FixedPrice.sol:
109:  ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);

OpenEdition.sol:
92: ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);

LPDA.sol:
85: ISaleFactory(factory).feeReceiver().transfer(fee);
86: temp.saleReceiver.transfer(totalSale - fee);

3- As per [OpenZeppelin’s (OZ) recommendation](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/6), “The guidelines are now to make it impossible for anyone to run initialize on an implementation contract, by adding an empty constructor with the initializer modifier. So the implementation contract gets initialized automatically upon deployment.”

It is also mentioned by the [OZ Wizard](https://wizard.openzeppelin.com/) since the UUPS vulnerability discovery: “Additionally, we modified the code generated by the Wizard 19 to include a constructor that automatically initializes the implementation when deployed.”

For example, this thwarts any attempts to frontrun the initialization of the following contract:
```
Escher721.sol:
32:  function initialize(
        address _creator,
        address _uri,
        string memory _name,
        string memory _symbol
    ) public initializer
```


