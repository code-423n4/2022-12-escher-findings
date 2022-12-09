# Gas

|   |issue   |
|---|---|
|1   |[1. internal functions not called by the contract should be removed](#1-internal-functions-not-called-by-the-contract-should-be-removed)|
|2   |[2. remove or change unneeded function](#2-remove-or-change-unneeded-function)|
|3   |[3. use unchecked if no over- or underflow is possible](#3-use-unchecked-if-no-over-or-underflow-is-possible)|
|4   |[4. change onlyOwner (admin external functions not for endusers) to payable](#4-change-onlyowner-admin-external-functions-not-for-endusers-to-payable)|
|5   |[5. storage pointer is cheaper than memory](#5-storage-pointer-is-cheaper-than-memory)|
|6   |[6. empty blocks should be removed or emit something](#6-empty-blocks-should-be-removed-or-emit-something)|


## 1. internal functions not called by the contract should be removed
```solidity
File: src/Escher721.sol
96:     function _burn(uint256 tokenId) internal override(ERC721Upgradeable) { // @audit is never used and can be removed
```

## 2. remove or change unneeded function
To save deployment size you can remove the tokenURI function, as the baseURI what it returns is public or change the visibility to private.

```solidity
File: src/uris/Base.sol
14:     function tokenURI(uint256) external view virtual returns (string memory) { // @audit remove function, baseURI is public
15:         return baseURI;
16:     }
```

## 3. use unchecked if no over or underflow is possible
If it's not possible that the equation will over- / underflow, especially if it was checked previously it's better to use an unchecked{} block to save gas.

```solidity
File: src/minters/FixedPrice.sol
65:         for (uint48 x = sale_.currentId + 1; x <= newId; x++) { // @audit use unchecked{++x;} at ende of loop instead of x++ here

File: src/minters/FixedPrice.sol
101:     function available() public view returns (uint256) {
102:         // @audit can be unchecked 
103:         // we have a check require(newId <= sale_.finalId, "TOO MANY"); in buy function 
104:         // and we have a check require(_sale.finalId > _sale.currentId, "FINAL ID BEFORE CURRENT"); in factory
105:         // so it's not possible to underflow, only if deployed not through the factory
106:         return sale.finalId - sale.currentId;
107:     }

File: src/minters/LPDA.sol
73:         for (uint256 x = temp.currentId + 1; x <= newId; x++) { // @audit use unchecked{++x;} at ende of loop instead of x++ here

File: src/minters/LPDA.sol
117:     function getPrice() public view returns (uint256) {
118:         Sale memory temp = sale; // @audit use storage pointer
119:         (uint256 start, uint256 end) = (temp.startTime, temp.endTime);
120:         if (block.timestamp < start) return type(uint256).max;
121:         if (temp.currentId == temp.finalId) return temp.finalPrice;
122:         // @audit can be unchecked
123:         // if end > block.timestamp the result for block.timestamp - start can underflow because of the check if (block.timestamp < start) above
124:         // end - start also can't underflow, as we have the check require(sale.endTime > sale.startTime, "INVALID END TIME") on the factory before deployment
125:         // so it's not possible to underflow, only if deployed not through the factory
126:         uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;
127:         return temp.startPrice - (temp.dropPerSecond * timeElapsed);
128:     }

File: src/minters/OpenEdition.sol
66:         for (uint24 x = temp.currentId + 1; x <= newId; x++) { // @audit use unchecked{++x;} at ende of loop instead of x++ here
```

Savings:
```git
 | src/minters/FixedPrice.sol:FixedPrice contract |                 |        |        |        |         |
 |------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                | Deployment Size |        |        |        |         |
-| 774471                                         | 3866            |        |        |        |         |
+| 754052                                         | 3764            |        |        |        |         |
 | Function Name                                  | min             | avg    | median | max    | # calls |
-| buy                                            | 953             | 62533  | 53556  | 288719 | 18      |
+| buy                                            | 953             | 62406  | 53474  | 287981 | 18      |

 | src/minters/FixedPriceFactory.sol:FixedPriceFactory contract |                 |        |        |        |         |
 |--------------------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                              | Deployment Size |        |        |        |         |
-| 1232310                                                      | 5988            |        |        |        |         |
+| 1211871                                                      | 5886            |        |        |        |         |

@@ -175,19 +173,19 @@ Test result: ok. 6 passed; 0 failed; finished in 2.37ms
 | src/minters/LPDA.sol:LPDA contract |                 |        |        |        |         |
 |------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                    | Deployment Size |        |        |        |         |
-| 1252237                            | 6103            |        |        |        |         |
+| 1243031                            | 6057            |        |        |        |         |
 | Function Name                      | min             | avg    | median | max    | # calls |
-| buy                                | 1149            | 125350 | 101393 | 295596 | 20      |
+| buy                                | 1149            | 125137 | 101265 | 294963 | 20      |
 | cancel                             | 475             | 1912   | 627    | 4635   | 3       |
-| getPrice                           | 1501            | 1501   | 1501   | 1501   | 1       |
+| getPrice                           | 1435            | 1435   | 1435   | 1435   | 1       |
 | initialize                         | 143516          | 143516 | 143516 | 143516 | 15      |
-| refund                             | 2148            | 7221   | 9174   | 9342   | 6       |
+| refund                             | 2082            | 7166   | 9142   | 9276   | 6       |

 | src/minters/LPDAFactory.sol:LPDAFactory contract |                 |        |        |        |         |
 |--------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                  | Deployment Size |        |        |        |         |
-| 1819191                                          | 8768            |        |        |        |         |
+| 1809978                                          | 8722            |        |        |        |         |

@@ -198,9 +196,9 @@ Test result: ok. 6 passed; 0 failed; finished in 2.37ms
 | src/minters/OpenEdition.sol:OpenEdition contract |                 |        |        |        |         |
 |--------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                  | Deployment Size |        |        |        |         |
-| 838419                                           | 4093            |        |        |        |         |
+| 830013                                           | 4051            |        |        |        |         |
 | Function Name                                    | min             | avg    | median | max    | # calls |
-| buy                                              | 952             | 36527  | 53643  | 53643  | 15      |
+| buy                                              | 952             | 36472  | 53561  | 53561  | 15      |

 | src/minters/OpenEditionFactory.sol:OpenEditionFactory contract |                 |        |        |        |         |
 |----------------------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                                | Deployment Size |        |        |        |         |
-| 1292500                                                        | 6196            |        |        |        |         |
+| 1284088                                                        | 6154            |        |        |        |         |
```

## 4. change onlyOwner (admin external functions not for endusers) to payable
If a function is only callable by special users like admins it makes sense to add payable. The compiler removes checks for msg.value, saving approximately 20 gas per function call.

```solidity
File: src/minters/FixedPrice.sol
50:     function cancel() external onlyOwner { // @audit use payable for protected functions not for endusers

File: src/minters/FixedPrice.sol
78:     function initialize(Sale memory _sale) public initializer { // @audit use payable for protected functions not for endusers

File: src/minters/FixedPriceFactory.sol
42:     function setFeeReceiver(address payable fees) public onlyOwner { // @audit use payable for protected functions not for endusers

File: src/minters/LPDA.sol
92:     function cancel() external onlyOwner { // @audit use payable for protected functions not for endusers

File: src/minters/LPDA.sol
110:     function initialize(Sale calldata _sale) public initializer { // @audit use payable for protected functions not for endusers

File: src/minters/LPDAFactory.sol
46:     function setFeeReceiver(address payable fees) public onlyOwner { // @audit use payable for protected functions not for endusers

File: src/minters/OpenEdition.sol
75:     function cancel() external onlyOwner { // @audit use payable for protected functions not for endusers

File: src/minters/OpenEdition.sol
82:     function initialize(Sale memory _sale) public initializer { // @audit use payable for protected functions not for endusers

File: src/minters/OpenEditionFactory.sol
42:     function setFeeReceiver(address payable fees) public onlyOwner { // @audit use payable for protected functions not for endusers

File: src/uris/Generative.sol
14:     function setScriptPiece(uint256 _id, string memory _data) external onlyOwner { // @audit use payable for protected functions not for endusers

File: src/uris/Generative.sol
19:     function setSeedBase() external onlyOwner {  // @audit use payable for protected functions not for endusers

File: src/Escher.sol
21:     function setURI(string memory _newuri) public onlyRole(DEFAULT_ADMIN_ROLE) { // @audit use payable for protected functions not for endusers

File: src/Escher.sol
27:     function addCreator(address _account) public onlyRole(CURATOR_ROLE) { // @audit use payable for protected functions not for endusers

File: src/Escher721.sol
32:     function initialize(
33:         address _creator,
34:         address _uri,
35:         string memory _name,
36:         string memory _symbol
37:     ) public initializer { // @audit use payable for protected functions not for endusers

File: src/Escher721.sol
51:     function mint(address to, uint256 tokenId) public virtual onlyRole(MINTER_ROLE) { // @audit use payable for protected functions not for endusers

File: src/Escher721.sol
57:     function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) { // @audit use payable for protected functions not for endusers

File: src/Escher721.sol
64:     function setDefaultRoyalty(
65:         address receiver,
66:         uint96 feeNumerator
67:     ) public onlyRole(DEFAULT_ADMIN_ROLE) { // @audit use payable for protected functions not for endusers

File: src/Escher721.sol
72:     function resetDefaultRoyalty() public onlyRole(DEFAULT_ADMIN_ROLE) { // @audit use payable for protected functions not for endusers
```

Savings:
```git
 | src/Escher.sol:Escher contract |                 |       |        |       |         |
 |--------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                | Deployment Size |       |        |       |         |
-| 1359780                        | 8785            |       |        |       |         |
+| 1398799                        | 8980            |       |        |       |         |
 | Function Name                  | min             | avg   | median | max   | # calls |
 | CREATOR_ROLE                   | 251             | 251   | 251    | 251   | 65      |
 | CURATOR_ROLE                   | 207             | 207   | 207    | 207   | 65      |
-| addCreator                     | 1284            | 52469 | 53372  | 53416 | 87      |
+| addCreator                     | 1260            | 52445 | 53348  | 53392 | 87      |

 | src/Escher721.sol:Escher721 contract |                 |        |        |        |         |
 |--------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                      | Deployment Size |        |        |        |         |
-| 1684964                              | 8448            |        |        |        |         |
+| 1742227                              | 8734            |        |        |        |         |
 | Function Name                        | min             | avg    | median | max    | # calls |
 | MINTER_ROLE                          | 239             | 239    | 239    | 239    | 34      |
 | grantRole                            | 27571           | 27571  | 27571  | 27571  | 34      |
 | hasRole                              | 2699            | 2699   | 2699   | 2699   | 42      |
-| initialize                           | 146919          | 156869 | 156869 | 166819 | 130     |
-| mint                                 | 25741           | 34047  | 25741  | 47641  | 87      |
-| updateURIDelegate                    | 6241            | 19258  | 19258  | 32275  | 2       |
+| initialize                           | 146895          | 156845 | 156845 | 166795 | 130     |
+| mint                                 | 25717           | 34023  | 25717  | 47617  | 87      |
+| updateURIDelegate                    | 6222            | 19236  | 19236  | 32251  | 2       |

 | src/Escher721Factory.sol:Escher721Factory contract |                 |        |        |        |         |
 |----------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                    | Deployment Size |        |        |        |         |
-| 2183516                                            | 10452           |        |        |        |         |
+| 2240819                                            | 10738           |        |        |        |         |
 | Function Name                                      | min             | avg    | median | max    | # calls |
-| createContract                                     | 258264          | 258264 | 258264 | 258264 | 65      |
+| createContract                                     | 258240          | 258240 | 258240 | 258240 | 65      |

 | src/minters/FixedPrice.sol:FixedPrice contract |                 |        |        |        |         |
 |------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                | Deployment Size |        |        |        |         |
-| 774471                                         | 3866            |        |        |        |         |
+| 769271                                         | 3840            |        |        |        |         |
 | Function Name                                  | min             | avg    | median | max    | # calls |
-| buy                                            | 953             | 62533  | 53556  | 288719 | 18      |
-| cancel                                         | 497             | 3868   | 1573   | 11831  | 4       |
-| initialize                                     | 3466            | 109036 | 117834 | 117834 | 13      |
+| buy                                            | 953             | 62496  | 53532  | 288503 | 18      |
+| cancel                                         | 473             | 3844   | 1549   | 11807  | 4       |
+| initialize                                     | 3442            | 109012 | 117810 | 117810 | 13      |
 | owner                                          | 320             | 1320   | 1320   | 2320   | 2       |
 | transferOwnership                              | 671             | 1671   | 1671   | 2671   | 2       |

@@ -163,47 +161,47 @@ Test result: ok. 6 passed; 0 failed; finished in 1.59ms
 | src/minters/FixedPriceFactory.sol:FixedPriceFactory contract |                 |        |        |        |         |
 |--------------------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                              | Deployment Size |        |        |        |         |
-| 1232310                                                      | 5988            |        |        |        |         |
+| 1240109                                                      | 6027            |        |        |        |         |
 | Function Name                                                | min             | avg    | median | max    | # calls |
-| createFixedSale                                              | 8795            | 163564 | 176462 | 176462 | 13      |
+| createFixedSale                                              | 8795            | 163542 | 176438 | 176438 | 13      |
 | feeReceiver                                                  | 347             | 2013   | 2347   | 2347   | 6       |
 | implementation                                               | 205             | 205    | 205    | 205    | 4       |
-| setFeeReceiver                                               | 2605            | 4089   | 4089   | 5573   | 2       |
+| setFeeReceiver                                               | 2581            | 4065   | 4065   | 5549   | 2       |
 | transferOwnership                                            | 2627            | 4910   | 4910   | 7193   | 2       |

 | src/minters/LPDA.sol:LPDA contract |                 |        |        |        |         |
 |------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                    | Deployment Size |        |        |        |         |
-| 1252237                            | 6103            |        |        |        |         |
+| 1247031                            | 6077            |        |        |        |         |
 | Function Name                      | min             | avg    | median | max    | # calls |
-| buy                                | 1149            | 125350 | 101393 | 295596 | 20      |
-| cancel                             | 475             | 1912   | 627    | 4635   | 3       |
+| buy                                | 1149            | 125291 | 101369 | 295380 | 20      |
+| cancel                             | 451             | 1888   | 603    | 4611   | 3       |
 | getPrice                           | 1501            | 1501   | 1501   | 1501   | 1       |
-| initialize                         | 143516          | 143516 | 143516 | 143516 | 15      |
+| initialize                         | 143492          | 143492 | 143492 | 143492 | 15      |
 | refund                             | 2148            | 7221   | 9174   | 9342   | 6       |

 | src/minters/LPDAFactory.sol:LPDAFactory contract |                 |        |        |        |         |
 |--------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                  | Deployment Size |        |        |        |         |
-| 1819191                                          | 8768            |        |        |        |         |
+| 1826984                                          | 8807            |        |        |        |         |
 | Function Name                                    | min             | avg    | median | max    | # calls |
-| createLPDASale                                   | 8818            | 192584 | 204836 | 204836 | 16      |
+| createLPDASale                                   | 8818            | 192562 | 204812 | 204812 | 16      |
 | feeReceiver                                      | 347             | 2061   | 2347   | 2347   | 7       |
-| setFeeReceiver                                   | 2605            | 4089   | 4089   | 5573   | 2       |
+| setFeeReceiver                                   | 2581            | 4065   | 4065   | 5549   | 2       |
 | transferOwnership                                | 2627            | 4910   | 4910   | 7193   | 2       |

 | src/minters/OpenEdition.sol:OpenEdition contract |                 |        |        |        |         |
 |--------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                  | Deployment Size |        |        |        |         |
-| 838419                                           | 4093            |        |        |        |         |
+| 833213                                           | 4067            |        |        |        |         |
 | Function Name                                    | min             | avg    | median | max    | # calls |
-| buy                                              | 952             | 36527  | 53643  | 53643  | 15      |
-| cancel                                           | 475             | 3086   | 1551   | 8767   | 4       |
+| buy                                              | 952             | 36511  | 53619  | 53619  | 15      |
+| cancel                                           | 451             | 3062   | 1527   | 8743   | 4       |
 | finalize                                         | 864             | 17848  | 6864   | 45816  | 3       |
-| initialize                                       | 3533            | 109142 | 117943 | 117943 | 13      |
+| initialize                                       | 3509            | 109118 | 117919 | 117919 | 13      |
 | owner                                            | 376             | 1376   | 1376   | 2376   | 2       |
 | transferOwnership                                | 649             | 1649   | 1649   | 2649   | 2       |

@@ -211,19 +209,19 @@ Test result: ok. 6 passed; 0 failed; finished in 1.59ms
 | src/minters/OpenEditionFactory.sol:OpenEditionFactory contract |                 |        |        |        |         |
 |----------------------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                                | Deployment Size |        |        |        |         |
-| 1292500                                                        | 6196            |        |        |        |         |
+| 1300300                                                        | 6235            |        |        |        |         |
 | Function Name                                                  | min             | avg    | median | max    | # calls |
-| createOpenEdition                                              | 8817            | 163665 | 176569 | 176569 | 13      |
+| createOpenEdition                                              | 8817            | 163642 | 176545 | 176545 | 13      |
 | feeReceiver                                                    | 325             | 1825   | 2325   | 2325   | 4       |
 | implementation                                                 | 205             | 205    | 205    | 205    | 5       |
-| setFeeReceiver                                                 | 2605            | 4089   | 4089   | 5573   | 2       |
+| setFeeReceiver                                                 | 2581            | 4065   | 4065   | 5549   | 2       |
 | transferOwnership                                              | 2627            | 4910   | 4910   | 7193   | 2       |
```

## 5. storage pointer is cheaper than memory
Reference types cached in memory cost more gas than using storage pointers, as new memory is allocated for these variables, copying data from storage to memory.

```git
--- a/2022-12-escher/src/minters/FixedPrice.sol
+++ b/2022-12-escher/src/minters/FixedPrice.sol
@@ -55,7 +55,7 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
     /// @notice buy from a fixed price sale after the sale starts
     /// @param _amount the amount of editions to buy
     function buy(uint256 _amount) external payable {
-        Sale memory sale_ = sale;
+        Sale storage sale_ = sale;
         IEscher721 nft = IEscher721(sale_.edition);
         require(block.timestamp >= sale_.startTime, "TOO SOON");
         require(_amount * sale_.price == msg.value, "WRONG PRICE");
@@ -66,9 +66,9 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
             nft.mint(msg.sender, x);
         }

-        sale.currentId = newId;
+        sale_.currentId = newId;

-        emit Buy(msg.sender, _amount, msg.value, sale);
+        emit Buy(msg.sender, _amount, msg.value, sale_);

         if (newId == sale_.finalId) _end(sale);
     }
@@ -104,7 +104,7 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {

     /// @notice Owner can cancel current sale
     /// @param _sale the sale info
-    function _end(Sale memory _sale) internal { // @audit use storage pointer
+    function _end(Sale storage _sale) internal { // @audit use storage pointer
         emit End(_sale);
         ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20); // @audit constants should be defined rather than using magic numbers
         selfdestruct(_sale.saleReceiver);

--- a/2022-12-escher/src/minters/LPDA.sol
+++ b/2022-12-escher/src/minters/LPDA.sol
@@ -97,11 +97,11 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {

     /// @notice allow a buyer to get a refund on the current price difference
     function refund() public {
-        Receipt memory r = receipts[msg.sender]; // @audit use storage pointer
+        Receipt storage r = receipts[msg.sender]; // @audit use storage pointer
         uint80 price = uint80(getPrice()) * r.amount;
         uint80 owed = r.balance - price;
         require(owed > 0, "NOTHING TO REFUND");
-        receipts[msg.sender].balance = price;
+        r.balance = price;
         payable(msg.sender).transfer(owed);
     }

--- a/2022-12-escher/src/minters/OpenEdition.sol
+++ b/2022-12-escher/src/minters/OpenEdition.sol
@@ -56,17 +56,17 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
     /// @param _amount the amount of editions to buy
     function buy(uint256 _amount) external payable {
         uint24 amount = uint24(_amount);
-        Sale memory temp = sale;
+        Sale storage temp = sale;
         IEscher721 nft = IEscher721(temp.edition);
         require(block.timestamp >= temp.startTime, "TOO SOON");
         require(block.timestamp < temp.endTime, "TOO LATE");
-        require(amount * sale.price == msg.value, "WRONG PRICE");
+        require(amount * temp.price == msg.value, "WRONG PRICE");
         uint24 newId = amount + temp.currentId;

         for (uint24 x = temp.currentId + 1; x <= newId; x++) {
             nft.mint(msg.sender, x);
         }
-        sale.currentId = newId;
+        temp.currentId = newId;

         emit Buy(msg.sender, amount, msg.value, temp);
     }
@@ -87,7 +87,7 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {

     /// @notice finish an open edition and payout the artist
     function finalize() public {
-        Sale memory temp = sale;
+        Sale storage temp = sale;
         require(block.timestamp >= temp.endTime, "TOO SOON");
         ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20); // @audit constants should be defined rather than using magic numbers
         _end(temp);
@@ -117,7 +117,7 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
         return sale.endTime > block.timestamp ? type(uint24).max : 0;
     }

-    function _end(Sale memory _sale) internal { // @audit use storage pointer
+    function _end(Sale storage _sale) internal { // @audit use storage pointer
         emit End(_sale);
         selfdestruct(_sale.saleReceiver);
     }
```

Savings:
```git
@@ -151,11 +149,11 @@ Test result: ok. 11 passed; 0 failed; finished in 24.93ms
 | src/minters/FixedPrice.sol:FixedPrice contract |                 |        |        |        |         |
 |------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                | Deployment Size |        |        |        |         |
-| 774471                                         | 3866            |        |        |        |         |
+| 733233                                         | 3660            |        |        |        |         |
 | Function Name                                  | min             | avg    | median | max    | # calls |
-| buy                                            | 953             | 62533  | 53556  | 288719 | 18      |
-| cancel                                         | 497             | 3868   | 1573   | 11831  | 4       |
-| initialize                                     | 3466            | 109036 | 117834 | 117834 | 13      |
+| buy                                            | 666             | 62640  | 53986  | 288981 | 18      |
+| cancel                                         | 497             | 3825   | 1573   | 11657  | 4       |
+| initialize                                     | 3466            | 109025 | 117822 | 117822 | 13      |

@@ -163,9 +161,9 @@ Test result: ok. 11 passed; 0 failed; finished in 24.93ms
 | src/minters/FixedPriceFactory.sol:FixedPriceFactory contract |                 |        |        |        |         |
 |--------------------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                              | Deployment Size |        |        |        |         |
-| 1232310                                                      | 5988            |        |        |        |         |
+| 1191033                                                      | 5782            |        |        |        |         |
 | Function Name                                                | min             | avg    | median | max    | # calls |
-| createFixedSale                                              | 8795            | 163564 | 176462 | 176462 | 13      |
+| createFixedSale                                              | 8795            | 163553 | 176450 | 176450 | 13      |
 | feeReceiver                                                  | 347             | 2013   | 2347   | 2347   | 6       |
 | implementation                                               | 205             | 205    | 205    | 205    | 4       |
 | setFeeReceiver                                               | 2605            | 4089   | 4089   | 5573   | 2       |

 @@ -175,19 +173,19 @@ Test result: ok. 11 passed; 0 failed; finished in 24.93ms
 | src/minters/LPDA.sol:LPDA contract |                 |        |        |        |         |
 |------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                    | Deployment Size |        |        |        |         |
-| 1252237                            | 6103            |        |        |        |         |
+| 1244631                            | 6065            |        |        |        |         |
 | Function Name                      | min             | avg    | median | max    | # calls |
 | buy                                | 1149            | 125350 | 101393 | 295596 | 20      |
 | cancel                             | 475             | 1912   | 627    | 4635   | 3       |
 | getPrice                           | 1501            | 1501   | 1501   | 1501   | 1       |
 | initialize                         | 143516          | 143516 | 143516 | 143516 | 15      |
-| refund                             | 2148            | 7221   | 9174   | 9342   | 6       |
+| refund                             | 2176            | 7195   | 9121   | 9289   | 6       |

@@ -198,12 +196,12 @@ Test result: ok. 11 passed; 0 failed; finished in 24.93ms
 | src/minters/OpenEdition.sol:OpenEdition contract |                 |        |        |        |         |
 |--------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                  | Deployment Size |        |        |        |         |
-| 838419                                           | 4093            |        |        |        |         |
+| 804187                                           | 3922            |        |        |        |         |
 | Function Name                                    | min             | avg    | median | max    | # calls |
-| buy                                              | 952             | 36527  | 53643  | 53643  | 15      |
-| cancel                                           | 475             | 3086   | 1551   | 8767   | 4       |
-| finalize                                         | 864             | 17848  | 6864   | 45816  | 3       |
-| initialize                                       | 3533            | 109142 | 117943 | 117943 | 13      |
+| buy                                              | 650             | 36492  | 53912  | 53912  | 15      |
+| cancel                                           | 475             | 3034   | 1551   | 8560   | 4       |
+| finalize                                         | 418             | 16188  | 2418   | 45730  | 3       |
+| initialize                                       | 3533            | 109107 | 117905 | 117905 | 13      |
 | owner                                            | 376             | 1376   | 1376   | 2376   | 2       |

@@ -211,9 +209,9 @@ Test result: ok. 11 passed; 0 failed; finished in 24.93ms
 | src/minters/OpenEditionFactory.sol:OpenEditionFactory contract |                 |        |        |        |         |
 |----------------------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                                | Deployment Size |        |        |        |         |
-| 1292500                                                        | 6196            |        |        |        |         |
+| 1258236                                                        | 6025            |        |        |        |         |
 | Function Name                                                  | min             | avg    | median | max    | # calls |
-| createOpenEdition                                              | 8817            | 163665 | 176569 | 176569 | 13      |
+| createOpenEdition                                              | 8817            | 163629 | 176531 | 176531 | 13      |

```

## 6. empty blocks should be removed or emit something
To save deployment size the code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract.
```solidity
File: src/Escher721.sol
25:     constructor() {}
```