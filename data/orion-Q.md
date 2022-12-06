The contracts at minters/* as such :
FixedPrice
LPDA
OpenEdition
OpenEditionFactory

Uses the transfer() methods to send ethers, within these methods a fixed amount of 2300 gas **at least** is always passed ,and also keeping in mind that the gas amount of EVM transactions can be changed anytime especially during hard forks, if this deployed already and that change happens the contract will then be using a very high amount of gas

so, it's better to use the builtin call() function  address(addr).call{value:amount}("") instead. 

for more see : https://eips.ethereum.org/EIPS/eip-1884#backwards-compatibility
