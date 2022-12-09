##### Gas Optimizations

Gas savings are estimated using the gas report of existing `forge test --gas-report` tests (the sum of all deployment costs and the sum of the costs of calling methods) and may vary depending on the implementation of the fix.

Sorted by average user savings.
|       | Issue                                                                  | Instances | Estimated gas(deployments) | Estimated gas(min method call) | Estimated gas(avg method call) | Estimated gas(max method call) |
| :---: | :--------------------------------------------------------------------- | :-------: | :------------------------: | :----------------------------: | :----------------------------: | :----------------------------: |
| **1** | Zero check before cloning ITokenUriDelegate                            |     1     |          -46 047           |             65 309             |             74 639             |             83 969             |
| **2** | Using of the memory keyword when storage pointer should be used        |     6     |          391 215           |             1 972              |             2 248              |              -178              |
| **3** | The event emits the same variable twice                                |     3     |           57 314           |               0                |             1 562              |             1 683              |
| **4** | Use local constants for roles                                          |     1     |          267 425           |              870               |              870               |              870               |
| **5** | Unchecking arithmetics operations that can't underflow/overflow        |     7     |          266 640           |               0                |              371               |             1 534              |
| **6** | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables |     3     |           8 055            |               0                |              181               |              210               |
| **7** | Use function args instead of memory variables                          |     2     |           48 856           |               0                |               61               |               75               |
| **8** | Do not initialize struct fields to zero                                |     3     |           87 106           |               0                |               0                |               0                |
| **9** | Storage var is used when local available                               |     2     |           3 655            |              -27               |              -37               |              -38               |
|   -   | **Overall gas savings**                                                |  **28**   |   **1 075 415 (1,22%)**    |       **68 136 (9,28%)**       |       **79 913 (4,39%)**       |       **88 142 (3,68%)**       |



**Total: 28 instances over 9 issues**

---

#### 1. **Zero check before cloning ITokenUriDelegate (1 instance)**

Deployment. Gas Saved: **-46 047**

Minimum Method Call. Gas Saved: **65 309**

Average Method Call. Gas Saved: **74 639**

Maximum Method Call. Gas Saved: **83 969**

**NOTE**: From tests `test/utils/EscherTest.sol:24`, I've come to the conclusion that using `address(0)` as the `_uri` argument might be acceptable. Which can lead to cloning the null address and calling initialization, which will cost the user a lot of gas depending on the blockchain network.

##### - src/Escher721Factory.sol:[35](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L35)

```diff
diff --git a/src/Escher721Factory.sol b/src/Escher721Factory.sol
index 9eb2771..59dbde2 100644
--- a/src/Escher721Factory.sol
+++ b/src/Escher721Factory.sol
@@ -32,10 +32,17 @@ contract Escher721Factory {
   32,  32:         require(escher.hasRole(escher.CREATOR_ROLE(), msg.sender), "NOT AUTHORIZED");
   33,  33: 
   34,  34:         escherClone = implementation.clone();
-  35     :-        address uriClone = _uri.clone();
-  36     :-
+       35:+        
+       36:+        address uriClone;
+       37:+        if (_uri == address(0)) {
+       38:+            uriClone = address(0);
+       39:+        } else {
+       40:+            uriClone = _uri.clone();
+       41:+            ITokenUriDelegate(uriClone).initialize(msg.sender);
+       42:+        }
+       43:+        
   37,  44:         Escher721(escherClone).initialize(msg.sender, uriClone, _name, _symbol);
-  38     :-        ITokenUriDelegate(uriClone).initialize(msg.sender);
+       45:+        
   39,  46: 
   40,  47:         emit NewEscher721Contract(msg.sender, escherClone, uriClone);
   41,  48:     }
```

---

#### 2. **Using of the memory keyword when storage pointer should be used (6 instances)**

Deployment. Gas Saved: **391 215**

Minimum Method Call. Gas Saved: **1 972**

Average Method Call. Gas Saved: **2 248**

Maximum Method Call. Gas Saved: **-178**

##### - src/minters/FixedPrice.sol:[58](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L58)

```diff
diff --git a/src/minters/FixedPrice.sol b/src/minters/FixedPrice.sol
index 4e2cb9e..363258b 100644
--- a/src/minters/FixedPrice.sol
+++ b/src/minters/FixedPrice.sol
@@ -55,7 +55,7 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
   55,  55:     /// @notice buy from a fixed price sale after the sale starts
   56,  56:     /// @param _amount the amount of editions to buy
   57,  57:     function buy(uint256 _amount) external payable {
-  58     :-        Sale memory sale_ = sale;
+       58:+        Sale storage sale_ = sale;
   59,  59:         IEscher721 nft = IEscher721(sale_.edition);
   60,  60:         require(block.timestamp >= sale_.startTime, "TOO SOON");
   61,  61:         require(_amount * sale_.price == msg.value, "WRONG PRICE");
```

##### - src/minters/LPDA.sol:[60](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L60), [100](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L100), [118](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L118)

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..f93c937 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -57,7 +57,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
   57,  57:     /// @param _amount the amount of editions to buy
   58,  58:     function buy(uint256 _amount) external payable {
   59,  59:         uint48 amount = uint48(_amount);
-  60     :-        Sale memory temp = sale;
+       60:+        Sale storage temp = sale;
   61,  61:         IEscher721 nft = IEscher721(temp.edition);
   62,  62:         require(block.timestamp >= temp.startTime, "TOO SOON");
   63,  63:         uint256 price = getPrice();
@@ -97,7 +97,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
   97,  97: 
   98,  98:     /// @notice allow a buyer to get a refund on the current price difference
   99,  99:     function refund() public {
- 100     :-        Receipt memory r = receipts[msg.sender];
+      100:+        Receipt storage r = receipts[msg.sender];
  101, 101:         uint80 price = uint80(getPrice()) * r.amount;
  102, 102:         uint80 owed = r.balance - price;
  103, 103:         require(owed > 0, "NOTHING TO REFUND");
@@ -115,7 +115,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
  115, 115: 
  116, 116:     /// @notice the price of the sale
  117, 117:     function getPrice() public view returns (uint256) {
- 118     :-        Sale memory temp = sale;
+      118:+        Sale storage temp = sale;
  119, 119:         (uint256 start, uint256 end) = (temp.startTime, temp.endTime);
  120, 120:         if (block.timestamp < start) return type(uint256).max;
  121, 121:         if (temp.currentId == temp.finalId) return temp.finalPrice;
```

##### - src/minters/OpenEdition.sol:[59](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L59), [90](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L90)

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..c26e276 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -56,7 +56,7 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
   56,  56:     /// @param _amount the amount of editions to buy
   57,  57:     function buy(uint256 _amount) external payable {
   58,  58:         uint24 amount = uint24(_amount);
-  59     :-        Sale memory temp = sale;
+       59:+        Sale storage temp = sale;
   60,  60:         IEscher721 nft = IEscher721(temp.edition);
   61,  61:         require(block.timestamp >= temp.startTime, "TOO SOON");
   62,  62:         require(block.timestamp < temp.endTime, "TOO LATE");
@@ -87,7 +87,7 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
   87,  87: 
   88,  88:     /// @notice finish an open edition and payout the artist
   89,  89:     function finalize() public {
-  90     :-        Sale memory temp = sale;
+       90:+        Sale storage temp = sale;
   91,  91:         require(block.timestamp >= temp.endTime, "TOO SOON");
   92,  92:         ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
   93,  93:         _end(temp);
```

---

#### 3. **The event emits the same variable twice. (3 instances)**

Deployment. Gas Saved: **57 314**

Minimum Method Call. Gas Saved: **0**

Average Method Call. Gas Saved: **1 562**

Maximum Method Call. Gas Saved: **1 683**

##### - src/minters/FixedPriceFactory.sol:[17](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPriceFactory.sol#L17)

```diff
diff --git a/src/minters/FixedPriceFactory.sol b/src/minters/FixedPriceFactory.sol
index a84e79e..a3b8ac6 100644
--- a/src/minters/FixedPriceFactory.sol
+++ b/src/minters/FixedPriceFactory.sol
@@ -14,7 +14,6 @@ contract FixedPriceFactory is Ownable {
   14,  14: 
   15,  15:     event NewFixedPriceContract(
   16,  16:         address indexed creator,
-  17     :-        address indexed edition,
   18,  17:         address indexed saleContract,
   19,  18:         FixedPrice.Sale saleInfo
   20,  19:     );
@@ -34,7 +33,7 @@ contract FixedPriceFactory is Ownable {
   34,  33:         clone = implementation.clone();
   35,  34:         FixedPrice(clone).initialize(_sale);
   36,  35: 
-  37     :-        emit NewFixedPriceContract(msg.sender, _sale.edition, clone, _sale);
+       36:+        emit NewFixedPriceContract(msg.sender, clone, _sale);
   38,  37:     }
   39,  38: 
   40,  39:     /// @notice set the fee receiver for fixed price editions
```

##### - src/minters/LPDAFactory.sol:[17](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDAFactory.sol#L17)

```diff
diff --git a/src/minters/LPDAFactory.sol b/src/minters/LPDAFactory.sol
index d044ca9..5807318 100644
--- a/src/minters/LPDAFactory.sol
+++ b/src/minters/LPDAFactory.sol
@@ -14,7 +14,6 @@ contract LPDAFactory is Ownable {
   14,  14: 
   15,  15:     event NewLPDAContract(
   16,  16:         address indexed _creator,
-  17     :-        address indexed _edition,
   18,  17:         address indexed _saleContract,
   19,  18:         LPDA.Sale _saleInfo
   20,  19:     );
@@ -38,7 +37,7 @@ contract LPDAFactory is Ownable {
   38,  37:         clone = implementation.clone();
   39,  38:         LPDA(clone).initialize(sale);
   40,  39: 
-  41     :-        emit NewLPDAContract(msg.sender, sale.edition, clone, sale);
+       40:+        emit NewLPDAContract(msg.sender, clone, sale);
   42,  41:     }
   43,  42: 
   44,  43:     /// @notice set the fee receiver for fixed price editions
```

##### - src/minters/OpenEditionFactory.sol:[17](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEditionFactory.sol#L17)

```diff
diff --git a/src/minters/OpenEditionFactory.sol b/src/minters/OpenEditionFactory.sol
index c03b307..0548c22 100644
--- a/src/minters/OpenEditionFactory.sol
+++ b/src/minters/OpenEditionFactory.sol
@@ -14,7 +14,6 @@ contract OpenEditionFactory is Ownable {
   14,  14: 
   15,  15:     event NewOpenEditionContract(
   16,  16:         address indexed creator,
-  17     :-        address indexed edition,
   18,  17:         address indexed saleContract,
   19,  18:         OpenEdition.Sale saleInfo
   20,  19:     );
@@ -34,7 +33,7 @@ contract OpenEditionFactory is Ownable {
   34,  33:         clone = implementation.clone();
   35,  34:         OpenEdition(clone).initialize(sale);
   36,  35: 
-  37     :-        emit NewOpenEditionContract(msg.sender, sale.edition, clone, sale);
+       36:+        emit NewOpenEditionContract(msg.sender, clone, sale);
   38,  37:     }
   39,  38: 
   40,  39:     /// @notice set the fee receiver for fixed price editions
```

---

#### 4. **Use local constants for roles (1 instance)**

**NOTE**: Use overriding a constant in the contract instead of calling an external function if the constant is unlikely to change in the future. Role constants usually follow the same pattern and rarely change.

Deployment. Gas Saved: **267 425**

Minimum Method Call. Gas Saved: **870**

Average Method Call. Gas Saved: **870**

Maximum Method Call. Gas Saved: **870**

##### - src/Escher721Factory.sol:[32](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/Escher721Factory.sol#L32)

```diff
diff --git a/src/Escher721Factory.sol b/src/Escher721Factory.sol
index 9eb2771..e8f87ea 100644
--- a/src/Escher721Factory.sol
+++ b/src/Escher721Factory.sol
@@ -9,6 +9,8 @@ import {ITokenUriDelegate} from "./interfaces/ITokenUriDelegate.sol";
    9,   9: contract Escher721Factory {
   10,  10:     using Clones for address;
   11,  11: 
+       12:+    bytes32 private constant CREATOR_ROLE = keccak256("CREATOR_ROLE");
+       13:+
   12,  14:     Escher public immutable escher;
   13,  15:     address public immutable implementation;
   14,  16: 
@@ -29,7 +31,7 @@ contract Escher721Factory {
   29,  31:         string memory _symbol,
   30,  32:         address _uri
   31,  33:     ) external returns (address escherClone) {
-  32     :-        require(escher.hasRole(escher.CREATOR_ROLE(), msg.sender), "NOT AUTHORIZED");
+       34:+        require(escher.hasRole(CREATOR_ROLE, msg.sender), "NOT AUTHORIZED");
   33,  35: 
   34,  36:         escherClone = implementation.clone();
   35,  37:         address uriClone = _uri.clone();
```

---

#### 5. **Unchecking arithmetics operations that can't underflow/overflow (7 instances)**

Deployment. Gas Saved: **266 640**

Minimum Method Call. Gas Saved: **0**

Average Method Call. Gas Saved: **371**

Maximum Method Call. Gas Saved: **1 534**

##### - src/minters/FixedPrice.sol:[65](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L65)

```diff
diff --git a/src/minters/FixedPrice.sol b/src/minters/FixedPrice.sol
index 4e2cb9e..be2a3d7 100644
--- a/src/minters/FixedPrice.sol
+++ b/src/minters/FixedPrice.sol
@@ -62,8 +62,11 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
   62,  62:         uint48 newId = uint48(_amount) + sale_.currentId;
   63,  63:         require(newId <= sale_.finalId, "TOO MANY");
   64,  64: 
-  65     :-        for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
+       65:+        for (uint48 x = sale_.currentId + 1; x <= newId;) {
   66,  66:             nft.mint(msg.sender, x);
+       67:+            unchecked {
+       68:+                ++x;
+       69:+            }
   67,  70:         }
   68,  71: 
   69,  72:         sale.currentId = newId;
```

##### - src/minters/LPDA.sol:[73](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L73)

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..8bc1680 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -70,8 +70,11 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
   70,  70:         receipts[msg.sender].amount += amount;
   71,  71:         receipts[msg.sender].balance += uint80(msg.value);
   72,  72: 
-  73     :-        for (uint256 x = temp.currentId + 1; x <= newId; x++) {
+       73:+        for (uint256 x = temp.currentId + 1; x <= newId;) {
   74,  74:             nft.mint(msg.sender, x);
+       75:+            unchecked {
+       76:+                ++x;
+       77:+            }
   75,  78:         }
   76,  79: 
   77,  80:         sale.currentId = newId;
```

##### - src/minters/OpenEdition.sol:[66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L66)

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..252fd65 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -63,8 +63,11 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
   63,  63:         require(amount * sale.price == msg.value, "WRONG PRICE");
   64,  64:         uint24 newId = amount + temp.currentId;
   65,  65: 
-  66     :-        for (uint24 x = temp.currentId + 1; x <= newId; x++) {
+       66:+        for (uint24 x = temp.currentId + 1; x <= newId;) {
   67,  67:             nft.mint(msg.sender, x);
+       68:+            unchecked {
+       69:+                ++x;
+       70:+            }
   68,  71:         }
   69,  72:         sale.currentId = newId;
   70,  73: 
```

##### - src/minters/FixedPrice.sol:[105](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L105), [112](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L112)

```diff
diff --git a/src/minters/FixedPrice.sol b/src/minters/FixedPrice.sol
index be2a3d7..ecd4296 100644
--- a/src/minters/FixedPrice.sol
+++ b/src/minters/FixedPrice.sol
@@ -102,14 +102,16 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
  102, 102: 
  103, 103:     /// @notice the number of NFTs still available
  104, 104:     function available() public view returns (uint256) {
- 105     :-        return sale.finalId - sale.currentId;
+      105:+        unchecked { return  sale.finalId - sale.currentId; }
  106, 106:     }
  107, 107: 
  108, 108:     /// @notice Owner can cancel current sale
  109, 109:     /// @param _sale the sale info
  110, 110:     function _end(Sale memory _sale) internal {
  111, 111:         emit End(_sale);
- 112     :-        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
+      112:+        uint256 _fee;
+      113:+        unchecked { _fee = address(this).balance / 20; }
+      114:+        ISaleFactory(factory).feeReceiver().transfer(_fee);
  113, 115:         selfdestruct(_sale.saleReceiver);
  114, 116:     }
  115, 117: }
```

##### - src/minters/LPDA.sol:[87](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L87)

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index 8bc1680..ec6ddad 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -84,7 +84,8 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
   84,  84:         if (newId == temp.finalId) {
   85,  85:             sale.finalPrice = uint80(price);
   86,  86:             uint256 totalSale = price * amountSold;
-  87     :-            uint256 fee = totalSale / 20;
+       87:+            uint256 fee;
+       88:+            unchecked { fee = totalSale / 20;}
   88,  89:             ISaleFactory(factory).feeReceiver().transfer(fee);
   89,  90:             temp.saleReceiver.transfer(totalSale - fee);
   90,  91:             _end();
```

##### - src/minters/OpenEdition.sol:[95](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L95)

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 252fd65..08a7ff2 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -92,7 +92,9 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
   92,  92:     function finalize() public {
   93,  93:         Sale memory temp = sale;
   94,  94:         require(block.timestamp >= temp.endTime, "TOO SOON");
-  95     :-        ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);
+       95:+        uint256 _fee;
+       96:+        unchecked { _fee = address(this).balance / 20; }
+       97:+        ISaleFactory(factory).feeReceiver().transfer(_fee);
   96,  98:         _end(temp);
   97,  99:     }
   98, 100: 
```

---

#### 6. **`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables**

Deployment. Gas Saved: **8 055**

Minimum Method Call. Gas Saved: **0**

Average Method Call. Gas Saved: **181**

Maximum Method Call. Gas Saved: **210**

##### - src/minters/LPDA.sol:[66](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L66), [70](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L70), [71](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L71)

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..3b35f16 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -63,12 +63,12 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
   63,  63:         uint256 price = getPrice();
   64,  64:         require(msg.value >= amount * price, "WRONG PRICE");
   65,  65: 
-  66     :-        amountSold += amount;
+       66:+        amountSold = amountSold + amount;
   67,  67:         uint48 newId = amount + temp.currentId;
   68,  68:         require(newId <= temp.finalId, "TOO MANY");
   69,  69: 
-  70     :-        receipts[msg.sender].amount += amount;
-  71     :-        receipts[msg.sender].balance += uint80(msg.value);
+       70:+        receipts[msg.sender].amount = receipts[msg.sender].amount + amount;
+       71:+        receipts[msg.sender].balance = receipts[msg.sender].balance + uint80(msg.value);
   72,  72: 
   73,  73:         for (uint256 x = temp.currentId + 1; x <= newId; x++) {
   74,  74:             nft.mint(msg.sender, x);
```

---

#### 7. **Use function args instead of memory variables (2 instances)**

Deployment. Gas Saved: **48 856**

Minimum Method Call. Gas Saved: **0**

Average Method Call. Gas Saved: **61**

Maximum Method Call. Gas Saved: **75**

##### - src/minters/LPDA.sol:[59](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L59)

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..966090d 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -56,18 +56,17 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
   56,  56:     /// @notice buy from a fixed price sale after the sale starts
   57,  57:     /// @param _amount the amount of editions to buy
   58,  58:     function buy(uint256 _amount) external payable {
-  59     :-        uint48 amount = uint48(_amount);
   60,  59:         Sale memory temp = sale;
   61,  60:         IEscher721 nft = IEscher721(temp.edition);
   62,  61:         require(block.timestamp >= temp.startTime, "TOO SOON");
   63,  62:         uint256 price = getPrice();
-  64     :-        require(msg.value >= amount * price, "WRONG PRICE");
+       63:+        require(msg.value >= _amount * price, "WRONG PRICE");
   65,  64: 
-  66     :-        amountSold += amount;
-  67     :-        uint48 newId = amount + temp.currentId;
+       65:+        amountSold += uint48(_amount);
+       66:+        uint48 newId = uint48(_amount) + temp.currentId;
   68,  67:         require(newId <= temp.finalId, "TOO MANY");
   69,  68: 
-  70     :-        receipts[msg.sender].amount += amount;
+       69:+        receipts[msg.sender].amount += uint48(_amount);
   71,  70:         receipts[msg.sender].balance += uint80(msg.value);
   72,  71: 
   73,  72:         for (uint256 x = temp.currentId + 1; x <= newId; x++) {
@@ -76,7 +75,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
   76,  75: 
   77,  76:         sale.currentId = newId;
   78,  77: 
-  79     :-        emit Buy(msg.sender, amount, msg.value, temp);
+       78:+        emit Buy(msg.sender, _amount, msg.value, temp);
   80,  79: 
   81,  80:         if (newId == temp.finalId) {
   82,  81:             sale.finalPrice = uint80(price);
```

##### - src/minters/OpenEdition.sol:[58](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L58)

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..c6797f3 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -55,20 +55,19 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
   55,  55:     /// @notice buy from a fixed price sale after the sale starts
   56,  56:     /// @param _amount the amount of editions to buy
   57,  57:     function buy(uint256 _amount) external payable {
-  58     :-        uint24 amount = uint24(_amount);
   59,  58:         Sale memory temp = sale;
   60,  59:         IEscher721 nft = IEscher721(temp.edition);
   61,  60:         require(block.timestamp >= temp.startTime, "TOO SOON");
   62,  61:         require(block.timestamp < temp.endTime, "TOO LATE");
-  63     :-        require(amount * sale.price == msg.value, "WRONG PRICE");
-  64     :-        uint24 newId = amount + temp.currentId;
+       62:+        require(_amount * sale.price == msg.value, "WRONG PRICE");
+       63:+        uint24 newId = uint24(_amount) + temp.currentId;
   65,  64: 
   66,  65:         for (uint24 x = temp.currentId + 1; x <= newId; x++) {
   67,  66:             nft.mint(msg.sender, x);
   68,  67:         }
   69,  68:         sale.currentId = newId;
   70,  69: 
-  71     :-        emit Buy(msg.sender, amount, msg.value, temp);
+       70:+        emit Buy(msg.sender, _amount, msg.value, temp);
   72,  71:     }
   73,  72: 
   74,  73:     /// @notice cancel a fixed price sale
```

---

#### 8. **Do not initialize struct fields to zero (3 instances)**

Deployment. Gas Saved: **87 106**

Minimum Method Call. Gas Saved: 0

Average Method Call. Gas Saved: 0

Maximum Method Call. Gas Saved: 0

##### - src/minters/FixedPrice.sol:[46](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L46)

```diff
diff --git a/src/minters/FixedPrice.sol b/src/minters/FixedPrice.sol
index 4e2cb9e..e35cc75 100644
--- a/src/minters/FixedPrice.sol
+++ b/src/minters/FixedPrice.sol
@@ -43,7 +43,8 @@ contract FixedPrice is Initializable, OwnableUpgradeable, ISale {
   43,  43:     constructor() {
   44,  44:         factory = msg.sender;
   45,  45:         _disableInitializers();
-  46     :-        sale = Sale(0, 0, address(0), type(uint96).max, payable(0), type(uint96).max);
+       46:+        sale.price = type(uint96).max;
+       47:+        sale.startTime = type(uint96).max;
   47,  48:     }
   48,  49: 
   49,  50:     /// @notice Owner can cancel current sale
```

##### - src/minters/LPDA.sol:[53](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L53)

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..bf7b614 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -46,11 +46,18 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
   46,  46:     constructor() {
   47,  47:         factory = msg.sender;
   48,  48:         _disableInitializers();
-  49     :-        uint48 x = type(uint48).max;
-  50     :-        uint80 y = type(uint80).max;
-  51     :-        uint96 z = type(uint96).max;
-  52     :-        address payable i = payable(address(0));
-  53     :-        sale = LPDA.Sale(x, x, i, y, y, y, z, i, z);
+       49:+        // uint48 x = type(uint48).max;
+       50:+        // uint80 y = type(uint80).max;
+       51:+        // uint96 z = type(uint96).max;
+       52:+        // address payable i = payable(address(0));
+       53:+        // sale = LPDA.Sale(x, x, i, y, y, y, z, i, z);
+       54:+        sale.currentId = type(uint48).max;
+       55:+        sale.finalId = type(uint48).max;
+       56:+        sale.startPrice = type(uint80).max;
+       57:+        sale.finalPrice = type(uint80).max;
+       58:+        sale.dropPerSecond = type(uint80).max;
+       59:+        sale.endTime = type(uint96).max;
+       60:+        sale.startTime = type(uint96).max;
   54,  61:     }
   55,  62: 
   56,  63:     /// @notice buy from a fixed price sale after the sale starts
```

##### - src/minters/OpenEdition.sol:[45](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L45)

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..b32bcae 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -42,14 +42,10 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
   42,  42:     constructor() {
   43,  43:         factory = msg.sender;
   44,  44:         _disableInitializers();
-  45     :-        sale = Sale(
-  46     :-            type(uint72).max,
-  47     :-            type(uint24).max,
-  48     :-            address(0),
-  49     :-            type(uint96).max,
-  50     :-            payable(0),
-  51     :-            type(uint96).max
-  52     :-        );
+       45:+        sale.price = type(uint72).max;
+       46:+        sale.currentId = type(uint24).max;
+       47:+        sale.startTime = type(uint96).max;
+       48:+        sale.endTime = type(uint96).max;
   53,  49:     }
   54,  50: 
   55,  51:     /// @notice buy from a fixed price sale after the sale starts
```

---

#### 9. **Storage var is used when local available (2 intances)**

Deployment. Gas Saved: **3 655**

Minimum Method Call. Gas Saved: **-27**

Average Method Call. Gas Saved: **-37**

Maximum Method Call. Gas Saved: **-38**

##### - src/minters/LPDA.sol:[112](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L112)

```diff
diff --git a/src/minters/LPDA.sol b/src/minters/LPDA.sol
index f52161c..dc1e11c 100644
--- a/src/minters/LPDA.sol
+++ b/src/minters/LPDA.sol
@@ -109,7 +109,7 @@ contract LPDA is Initializable, OwnableUpgradeable, ISale {
  109, 109:     /// @param _sale the sale info
  110, 110:     function initialize(Sale calldata _sale) public initializer {
  111, 111:         sale = _sale;
- 112     :-        _transferOwnership(sale.saleReceiver);
+      112:+        _transferOwnership(_sale.saleReceiver);
  113, 113:         emit Start(_sale);
  114, 114:     }
  115, 115: 
```

##### - src/minters/OpenEdition.sol:[84](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L84)

```diff
diff --git a/src/minters/OpenEdition.sol b/src/minters/OpenEdition.sol
index 004f7fd..c28e72b 100644
--- a/src/minters/OpenEdition.sol
+++ b/src/minters/OpenEdition.sol
@@ -81,7 +81,7 @@ contract OpenEdition is Initializable, OwnableUpgradeable, ISale {
   81,  81:     /// @param _sale the sale info
   82,  82:     function initialize(Sale memory _sale) public initializer {
   83,  83:         sale = _sale;
-  84     :-        _transferOwnership(sale.saleReceiver);
+       84:+        _transferOwnership(_sale.saleReceiver);
   85,  85:         emit Start(_sale);
   86,  86:     }
   87,  87: 
```

---

#### 99. **Total gas change**

All optimizations merged into one commit

Deployment. Gas Saved: **1 075 415**

Minimum Method Call. Gas Saved: **68 136**

Average Method Call. Gas Saved: **79 913**

Maximum Method Call. Gas Saved: **88 142**

```diff
diff --git a/original.txt b/total-gas.txt
index be349b9..4b68940 100644
--- a/original.txt
+++ b/total-gas.txt
 | src/Escher.sol:Escher contract |                 |       |        |       |         |
 |--------------------------------|-----------------|-------|--------|-------|---------|
 | Deployment Cost                | Deployment Size |       |        |       |         |
 | 1359736                        | 8785            |       |        |       |         |
 | Function Name                  | min             | avg   | median | max   | # calls |
-| CREATOR_ROLE                   | 251             | 251   | 251    | 251   | 65      |
 | CURATOR_ROLE                   | 207             | 207   | 207    | 207   | 65      |
 | addCreator                     | 1284            | 52469 | 53372  | 53416 | 87      |
 | balanceOf                      | 700             | 1776  | 2700   | 2700  | 13      |
@@ -135,26 +134,26 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | MINTER_ROLE                          | 239             | 239    | 239    | 239    | 34      |
 | grantRole                            | 27571           | 27571  | 27571  | 27571  | 34      |
 | hasRole                              | 2699            | 2699   | 2699   | 2699   | 42      |
-| initialize                           | 146919          | 156869 | 156869 | 166819 | 130     |
+| initialize                           | 146919          | 146919 | 146919 | 146919 | 130     |
 | mint                                 | 25741           | 34047  | 25741  | 47641  | 87      |
-| updateURIDelegate                    | 6241            | 19258  | 19258  | 32275  | 2       |
+| updateURIDelegate                    | 5001            | 18638  | 18638  | 32275  | 2       |
 
 
 | src/Escher721Factory.sol:Escher721Factory contract |                 |        |        |        |         |
 |----------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                    | Deployment Size |        |        |        |         |
-| 2183516                                            | 10452           |        |        |        |         |
+| 2162292                                            | 10341           |        |        |        |         |
 | Function Name                                      | min             | avg    | median | max    | # calls |
-| createContract                                     | 258264          | 258264 | 258264 | 258264 | 65      |
+| createContract                                     | 193576          | 193576 | 193576 | 193576 | 65      |
 
 
 | src/minters/FixedPrice.sol:FixedPrice contract |                 |        |        |        |         |
 |------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                | Deployment Size |        |        |        |         |
-| 774471                                         | 3866            |        |        |        |         |
+| 734322                                         | 3630            |        |        |        |         |
 | Function Name                                  | min             | avg    | median | max    | # calls |
-| buy                                            | 953             | 62533  | 53556  | 288719 | 18      |
-| cancel                                         | 497             | 3868   | 1573   | 11831  | 4       |
+| buy                                            | 666             | 62406  | 53721  | 288196 | 18      |
+| cancel                                         | 497             | 3860   | 1573   | 11799  | 4       |
 | initialize                                     | 3466            | 109036 | 117834 | 117834 | 13      |
 | owner                                          | 320             | 1320   | 1320   | 2320   | 2       |
 | transferOwnership                              | 671             | 1671   | 1671   | 2671   | 2       |
@@ -163,9 +162,9 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | src/minters/FixedPriceFactory.sol:FixedPriceFactory contract |                 |        |        |        |         |
 |--------------------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                              | Deployment Size |        |        |        |         |
-| 1232310                                                      | 5988            |        |        |        |         |
+| 1186912                                                      | 5726            |        |        |        |         |
 | Function Name                                                | min             | avg    | median | max    | # calls |
-| createFixedSale                                              | 8795            | 163564 | 176462 | 176462 | 13      |
+| createFixedSale                                              | 8795            | 163046 | 175901 | 175901 | 13      |
 | feeReceiver                                                  | 347             | 2013   | 2347   | 2347   | 6       |
 | implementation                                               | 205             | 205    | 205    | 205    | 4       |
 | setFeeReceiver                                               | 2605            | 4089   | 4089   | 5573   | 2       |
@@ -175,21 +174,21 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | src/minters/LPDA.sol:LPDA contract |                 |        |        |        |         |
 |------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                    | Deployment Size |        |        |        |         |
-| 1252237                            | 6103            |        |        |        |         |
+| 1146584                            | 5508            |        |        |        |         |
 | Function Name                      | min             | avg    | median | max    | # calls |
-| buy                                | 1149            | 125350 | 101393 | 295596 | 20      |
-| cancel                             | 475             | 1912   | 627    | 4635   | 3       |
-| getPrice                           | 1501            | 1501   | 1501   | 1501   | 1       |
-| initialize                         | 143516          | 143516 | 143516 | 143516 | 15      |
-| refund                             | 2148            | 7221   | 9174   | 9342   | 6       |
+| buy                                | 644             | 124891 | 101067 | 294844 | 20      |
+| cancel                             | 475             | 1916   | 627    | 4648   | 3       |
+| getPrice                           | 1268            | 1268   | 1268   | 1268   | 1       |
+| initialize                         | 143543          | 143543 | 143543 | 143543 | 15      |
+| refund                             | 1943            | 6997   | 8916   | 9134   | 6       |
 
 
 | src/minters/LPDAFactory.sol:LPDAFactory contract |                 |        |        |        |         |
 |--------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                  | Deployment Size |        |        |        |         |
-| 1819191                                          | 8768            |        |        |        |         |
+| 1708213                                          | 8147            |        |        |        |         |
 | Function Name                                    | min             | avg    | median | max    | # calls |
-| createLPDASale                                   | 8818            | 192584 | 204836 | 204836 | 16      |
+| createLPDASale                                   | 8818            | 192084 | 204302 | 204302 | 16      |
 | feeReceiver                                      | 347             | 2061   | 2347   | 2347   | 7       |
 | setFeeReceiver                                   | 2605            | 4089   | 4089   | 5573   | 2       |
 | transferOwnership                                | 2627            | 4910   | 4910   | 7193   | 2       |
@@ -198,12 +197,12 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | src/minters/OpenEdition.sol:OpenEdition contract |                 |        |        |        |         |
 |--------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                  | Deployment Size |        |        |        |         |
-| 838419                                           | 4093            |        |        |        |         |
+| 818701                                           | 3951            |        |        |        |         |
 | Function Name                                    | min             | avg    | median | max    | # calls |
-| buy                                              | 952             | 36527  | 53643  | 53643  | 15      |
-| cancel                                           | 475             | 3086   | 1551   | 8767   | 4       |
-| finalize                                         | 864             | 17848  | 6864   | 45816  | 3       |
-| initialize                                       | 3533            | 109142 | 117943 | 117943 | 13      |
+| buy                                              | 644             | 36358  | 53725  | 53725  | 15      |
+| cancel                                           | 475             | 3079   | 1551   | 8741   | 4       |
+| finalize                                         | 418             | 16242  | 2418   | 45891  | 3       |
+| initialize                                       | 3533            | 109109 | 117908 | 117908 | 13      |
 | owner                                            | 376             | 1376   | 1376   | 2376   | 2       |
 | transferOwnership                                | 649             | 1649   | 1649   | 2649   | 2       |
 
@@ -211,9 +210,9 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | src/minters/OpenEditionFactory.sol:OpenEditionFactory contract |                 |        |        |        |         |
 |----------------------------------------------------------------|-----------------|--------|--------|--------|---------|
 | Deployment Cost                                                | Deployment Size |        |        |        |         |
-| 1292500                                                        | 6196            |        |        |        |         |
+| 1267553                                                        | 6028            |        |        |        |         |
 | Function Name                                                  | min             | avg    | median | max    | # calls |
-| createOpenEdition                                              | 8817            | 163665 | 176569 | 176569 | 13      |
+| createOpenEdition                                              | 8817            | 163114 | 175973 | 175973 | 13      |
 | feeReceiver                                                    | 325             | 1825   | 2325   | 2325   | 4       |
 | implementation                                                 | 205             | 205    | 205    | 205    | 5       |
 | setFeeReceiver                                                 | 2605            | 4089   | 4089   | 5573   | 2       |
@@ -231,7 +230,7 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | test/Escher721.t.sol:Escher721Test contract |                 |      |        |      |         |
 |---------------------------------------------|-----------------|------|--------|------|---------|
 | Deployment Cost                             | Deployment Size |      |        |      |         |
-| 5969831                                     | 27413           |      |        |      |         |
+| 5947604                                     | 27302           |      |        |      |         |
 | Function Name                               | min             | avg  | median | max  | # calls |
 | onERC1155Received                           | 1023            | 1023 | 1023   | 1023 | 6       |
 
@@ -239,7 +238,7 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | test/FixedPrice.t.sol:FactoryTest contract |                 |      |        |      |         |
 |--------------------------------------------|-----------------|------|--------|------|---------|
 | Deployment Cost                            | Deployment Size |      |        |      |         |
-| 7683312                                    | 35968           |      |        |      |         |
+| 7608595                                    | 35595           |      |        |      |         |
 | Function Name                              | min             | avg  | median | max  | # calls |
 | onERC1155Received                          | 1023            | 1023 | 1023   | 1023 | 18      |
 
@@ -247,7 +246,7 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | test/FixedPrice.t.sol:FixedPriceImplementationTest contract |                 |      |        |      |         |
 |-------------------------------------------------------------|-----------------|------|--------|------|---------|
 | Deployment Cost                                             | Deployment Size |      |        |      |         |
-| 7607795                                                     | 35591           |      |        |      |         |
+| 7533078                                                     | 35218           |      |        |      |         |
 | Function Name                                               | min             | avg  | median | max  | # calls |
 | onERC1155Received                                           | 1023            | 1023 | 1023   | 1023 | 12      |
 
@@ -255,7 +254,7 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | test/FixedPrice.t.sol:FixedPriceTest contract |                 |      |        |      |         |
 |-----------------------------------------------|-----------------|------|--------|------|---------|
 | Deployment Cost                               | Deployment Size |      |        |      |         |
-| 8147426                                       | 38285           |      |        |      |         |
+| 8072716                                       | 37912           |      |        |      |         |
 | Function Name                                 | min             | avg  | median | max  | # calls |
 | onERC1155Received                             | 1001            | 1001 | 1001   | 1001 | 33      |
 | receive                                       | 55              | 55   | 55     | 55   | 3       |
@@ -264,7 +263,7 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | test/LPDA.t.sol:LPDAFactoryTest contract |                 |      |        |      |         |
 |------------------------------------------|-----------------|------|--------|------|---------|
 | Deployment Cost                          | Deployment Size |      |        |      |         |
-| 8321499                                  | 39154           |      |        |      |         |
+| 8174866                                  | 38422           |      |        |      |         |
 | Function Name                            | min             | avg  | median | max  | # calls |
 | onERC1155Received                        | 1023            | 1023 | 1023   | 1023 | 18      |
 
@@ -272,7 +271,7 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | test/LPDA.t.sol:LPDATest contract |                 |     |        |     |         |
 |-----------------------------------|-----------------|-----|--------|-----|---------|
 | Deployment Cost                   | Deployment Size |     |        |     |         |
-| 9040673                           | 42744           |     |        |     |         |
+| 8894030                           | 42012           |     |        |     |         |
 | Function Name                     | min             | avg | median | max | # calls |
 | onERC1155Received                 | 979             | 979 | 979    | 979 | 42      |
 | receive                           | 55              | 55  | 55     | 55  | 8       |
@@ -281,7 +280,7 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | test/OpenEdition.t.sol:OpenEditionFactoryTest contract |                 |      |        |      |         |
 |--------------------------------------------------------|-----------------|------|--------|------|---------|
 | Deployment Cost                                        | Deployment Size |      |        |      |         |
-| 7751610                                                | 36309           |      |        |      |         |
+| 7695731                                                | 36030           |      |        |      |         |
 | Function Name                                          | min             | avg  | median | max  | # calls |
 | onERC1155Received                                      | 1001            | 1001 | 1001   | 1001 | 18      |
 
@@ -289,7 +288,7 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | test/OpenEdition.t.sol:OpenEditionImplementation contract |                 |      |        |      |         |
 |-----------------------------------------------------------|-----------------|------|--------|------|---------|
 | Deployment Cost                                           | Deployment Size |      |        |      |         |
-| 7726980                                                   | 36186           |      |        |      |         |
+| 7671092                                                   | 35907           |      |        |      |         |
 | Function Name                                             | min             | avg  | median | max  | # calls |
 | onERC1155Received                                         | 1001            | 1001 | 1001   | 1001 | 15      |
 
@@ -297,7 +296,7 @@ Test result: [32mok[0m. 14 passed; 0 failed; finished in 27.22ms
 | test/OpenEdition.t.sol:OpenEditionTest contract |                 |     |        |     |         |
 |-------------------------------------------------|-----------------|-----|--------|-----|---------|
 | Deployment Cost                                 | Deployment Size |     |        |     |         |
-| 8184886                                         | 38472           |     |        |     |         |
+| 8128996                                         | 38193           |     |        |     |         |
 | Function Name                                   | min             | avg | median | max | # calls |
 | onERC1155Received                               | 979             | 979 | 979    | 979 | 33      |
 | receive                                         | 55              | 55  | 55     | 55  | 1       |
```