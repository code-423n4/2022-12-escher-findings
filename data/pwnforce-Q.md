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