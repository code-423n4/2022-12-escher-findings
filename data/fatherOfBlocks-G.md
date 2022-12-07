**src/uris/Base.sol**
- L8/14 - the variable in storage baseURI is public, therefore it has its own getter, when creating a particular getter, the same getter is being created twice. The solution to this is either to remove the function from L14 or to set the variable as private.


**src/uris/Generative.sol**
- L15/20/28 - Instead of using a require you can use ifs and an error custom, this would generate a lower cost of gas.


**src/Escher721Factory.sol**
- L30/31/32 - Instead of using a require you can use ifs and an error custom, this would generate a lower cost of gas.


**src/minters/OpenEditionFactory.sol**
- L30/31/32 - Instead of using a require you can use ifs and an error custom, this would generate a lower cost of gas.


**src/minters/LPDAFactory.sol**
- L30/31/32/33/34/35/36 - Instead of using a require you can use ifs and an error custom, this would generate a lower cost of gas.

- L35/36 - It is less expensive to validate that uint != 0 than to validate uint > 0

- L31 - Use assembly to check for address(0), Saves 6 gas per instance.


**src/minters/FixedPrice.sol**
- L51/60/61/63 - Instead of using a require you can use ifs and an error custom, this would generate a lower cost of gas.


**src/minters/OpenEdition.sol**
- L61/62/63/76/91 - Instead of using a require you can use ifs and an error custom, this would generate a lower cost of gas.


**src/minters/LPDA.sol**
- L36 - It is not necessary to create a variable and set a value when we want to set its default value, this is so since when creating the variable it already has that value.

- L62/64/68/93/103 - Instead of using a require you can use ifs and an error custom, this would generate a lower cost of gas.

- L103 - It is less expensive to validate that uint != 0 than to validate uint > 0
