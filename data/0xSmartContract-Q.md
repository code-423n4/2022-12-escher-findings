## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]| Use `Ownable2StepUpgradeable` instead of ` OwnableUpgradeable ` contract| 3 |
|[L-02]| Use ```safeTransferOwnership``` instead of ```transferOwnership``` function | 1 |
|[L-03]| Owner can renounce Ownership| 3 |
|[L-04]| Critical Address Changes Should Use Two-step Procedure| 4 |
|[L-05]| Initial value check is missing in Set Functions| 5 |
|[L-06]| `setFeeReceiver` value check is missing in critical Set Functions | 3 |
|[L-07]|Protect your NFT from copying in POW forks |  |

Total 7 issues


### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01]|Insufficient coverage||
| [N-02] |Test arguments and actual contract arguments are incompatible  |1|
| [N-03] |Constant values such as a call to keccak256(), should used to immutable rather than constant| 4 |
| [N-04] |For functions, follow Solidity standard naming conventions | 1 |
| [N-05] |`Function writing` that does not comply with the `Solidity Style Guide`| All contracts|
| [N-06] |Lack of event emission after critical `initialize()` function  | 1 |
| [N-07] |Add a timelock to critical functions| 3 |
| [N-08] |`Empty blocks` should be _removed_ or _Emit_ something  | 1 |
| [N-09] |NatSpec comments should be increased in contracts | All contracts |
| [N-10] |Floating pragma| All contracts |
| [N-11] |Loss of precision due to rounding |3 |
| [N-12] |Add parameter to Event-Emit | 3 |
| [N-13] |Omissions in Events |3 |
| [N-14] |Lack of event emission after critical `initialize()` functions| 1 |
| [N-15] |NatSpec is missing  | 11 |

Total 15 issues


### Suggestions
| Number | Suggestion Details |
|:--:|:-------|
| [S-01] |Mark visibility of initialize(...) functions as `external` |
| [S-02] |Generate perfect code headers every time |

Total 2 suggestions

### [L-01] Use `Ownable2StepUpgradeable` instead of ` OwnableUpgradeable ` contract

**Context:**
[FixedPrice.sol#L10](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L10)
[LPDA.sol#L10](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L10)
[OpenEdition.sol#L10](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L10)

**Description:**
```transferOwnership``` function is used to change Ownership from ```OwnableUpgradeable.sol```.

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has  ` transferOwnership` function ,  use it is more secure due to 2-stage ownership transfer.

[Ownable2StepUpgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/access/Ownable2StepUpgradeable.sol)

### [L-02] Use ```safeTransferOwnership``` instead of ```transferOwnership``` function

**Context:**
[FixedPriceFactory.sol#L9](https://github.com/code-423n4/2022-12escher/blob/main/src/minters/FixedPriceFactory.sol#L9)
[LPDAFactory.sol#L9](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L9)
[OpenEditionFactory.sol#L9](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L9)

**Description:**
```transferOwnership``` function is used to change Ownership from ```Ownable.sol```.

Use a 2 structure transferOwnership which is safer. 
```safeTransferOwnership```,  use it is more secure due to 2-stage ownership transfer.

**Recommendation:**
Use ``Ownable2Step.sol``
[Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)

### [L-03] Owner can renounce Ownership

**Context:**
[FixedPriceFactory.sol#L9](https://github.com/code-423n4/2022-12escher/blob/main/src/minters/FixedPriceFactory.sol#L9)
[LPDAFactory.sol#L9](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDAFactory.sol#L9)
[OpenEditionFactory.sol#L9](https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEditionFactory.sol#L9)

**Description:**
Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

`onlyOwner` function;
```js
function setFeeReceiver(address payable fees) public onlyOwner 
```

**Recommendation:**
We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

### [L-04] Critical Address Changes Should Use Two-step Procedure

The critical procedures should be two step process.

4 results 3 files
```solidity
src/minters/FixedPriceFactory.sol:
  48:     function setFeeReceiver(address payable fees) public onlyOwner {
  49          feeReceiver = fees;

src/minters/LPDAFactory.sol:
  46:     function setFeeReceiver(address payable fees) public onlyOwner {
  47          feeReceiver = fees;

src/minters/OpenEditionFactory.sol:
  42:     function setFeeReceiver(address payable fees) public onlyOwner {
  43          feeReceiver = fees;

```

Recommended Mitigation Steps
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-05] Initial value check is missing in Set Functions

**Context:**
```solidity
5 results 4 files
src/minters/FixedPriceFactory.sol:
  48:     function setFeeReceiver(address payable fees) public onlyOwner {
  49          feeReceiver = fees;

src/minters/LPDAFactory.sol:
  46:     function setFeeReceiver(address payable fees) public onlyOwner {
  47          feeReceiver = fees;

src/minters/OpenEditionFactory.sol:
  42:     function setFeeReceiver(address payable fees) public onlyOwner {
  43          feeReceiver = fees;

src/Escher721.sol:
  57:     function updateURIDelegate(address _uriDelegate) public onlyRole(URI_SETTER_ROLE) {
  58:         tokenUriDelegate = _uriDelegate;

  64:     function setDefaultRoyalty(
  65:         address receiver,
  66:         uint96 feeNumerator
  67:     ) public onlyRole(DEFAULT_ADMIN_ROLE) {
  68:         _setDefaultRoyalty(receiver, feeNumerator);
  69      }

```
Checking whether the current value and the new value are the same should be added

### [L-06] `setFeeReceiver` value check is missing in critical Set Functions

if `onlyOwner` enter to 0 address in `setFeeReceiver` (it is imposible) , platform lose Money , so should add to address check require in function

```solidity

3 results - 3 files

src/minters/FixedPriceFactory.sol:
  48:     function setFeeReceiver(address payable fees) public onlyOwner {
  49:         feeReceiver = fees;
  50:     }
  51  }

src/minters/LPDAFactory.sol:
  46:     function setFeeReceiver(address payable fees) public onlyOwner {
  47:         feeReceiver = fees;
  48:     }
  49  }

src/minters/OpenEditionFactory.sol:
  42:     function setFeeReceiver(address payable fees) public onlyOwner {
  43:         feeReceiver = fees;
  44:     }
  45  }

```

### [L-07] Protect your NFT from copying in POW forks

Ethereum has performed the long-awaited “merge” that will dramatically reduce the environmental impact of the network

There may be forked versions of Ethereum, which could cause confusion and lead to scams as duplicated NFT assets enter the market.

If the Ethereum Merge, which took place in September 2022, results in the Blockchain splitting into two Blockchains due to the 'THE DAO' attack in 2016, this could result in duplication of immutable tokens (NFTs).

In any case, duplicate NFTs will exist due to the ETH proof-of-work chain and other potential forks, and there’s likely to be some level of confusion around which assets are ‘official’ or ‘authentic.’

Even so, there could be a frenzy for these copies, as NFT owners attempt to flip the proof-of-work versions of their valuable tokens.

As ETHPOW and any other forks spin off of the Ethereum mainnet, they will yield duplicate versions of Ethereum’s NFTs. An NFT is simply a blockchain token, and it can work as a deed of ownership to digital items like artwork and collectibles. A forked Ethereum chain will thus have duplicated deeds that point to the same tokenURI

About Merge Replay Attack: 
https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

The most important part here is NFT's tokenURI detail. If the update I suggested below is not done, Duplicate NFTs will appear as a result, potentially leading to confusion and scams.

```solidity
src/Escher721.sol:
  77      /// @param _tokenId the token ID to get the URI for
  78:     function tokenURI(
  79          uint256 _tokenId

```
Recommendation code:
```solidity
src/Escher721.sol:
  77      /// @param _tokenId the token ID to get the URI for
  78:     function tokenURI(
                 if(block.chainid != 1) { 
                     revert(); 
                 }
  79          uint256 _tokenId

```

### [N-01] Insufficient coverage

**Description:**
The test coverage rate of the project is 72%. Testing all functions is best practice in terms of security criteria.

```js

| File                               | % Lines          | % Statements     | % Branches      | % Funcs        |
|------------------------------------|------------------|------------------|-----------------|----------------|
| src/Escher.sol                     | 80.00% (8/10)    | 80.00% (8/10)    | 100.00% (2/2)   | 71.43% (5/7)   |
| src/Escher721.sol                  | 15.38% (2/13)    | 15.38% (2/13)    | 100.00% (0/0)   | 25.00% (2/8)   |
| src/Escher721Factory.sol           | 0.00% (0/6)      | 0.00% (0/7)      | 0.00% (0/2)     | 0.00% (0/1)    |
| src/minters/FixedPrice.sol         | 82.61% (19/23)   | 86.21% (25/29)   | 100.00% (10/10) | 50.00% (4/8)   |
| src/minters/FixedPriceFactory.sol  | 100.00% (7/7)    | 100.00% (7/7)    | 66.67% (4/6)    | 100.00% (2/2)  |
| src/minters/LPDA.sol               | 91.11% (41/45)   | 91.53% (54/59)   | 87.50% (14/16)  | 60.00% (6/10)  |
| src/minters/LPDAFactory.sol        | 100.00% (11/11)  | 100.00% (11/11)  | 57.14% (8/14)   | 100.00% (2/2)  |
| src/minters/OpenEdition.sol        | 81.48% (22/27)   | 84.85% (28/33)   | 100.00% (10/10) | 50.00% (5/10)  |
| src/minters/OpenEditionFactory.sol | 100.00% (7/7)    | 100.00% (7/7)    | 66.67% (4/6)    | 100.00% (2/2)  |
| src/uris/Base.sol                  | 0.00% (0/3)      | 0.00% (0/3)      | 100.00% (0/0)   | 0.00% (0/3)    |
| src/uris/Generative.sol            | 0.00% (0/8)      | 0.00% (0/8)      | 0.00% (0/6)     | 0.00% (0/3)    |
| src/uris/Unique.sol                | 0.00% (0/1)      | 0.00% (0/1)      | 100.00% (0/0)   | 0.00% (0/1)    |

```
Due to its capacity, test coverage is expected to be 100%.

### [N-02] Test arguments and actual contract arguments are incompatible

`FixedPrice.sol.constructor()`  test file is  `test/FixedPrice.t.sol.setUp()` but used arguments is different.
Expected value of arguments should be same.

```solidity
src/minters/FixedPrice.sol:
  42      /// @custom:oz-upgrades-unsafe-allow constructor
  43:     constructor() {
  44:         factory = msg.sender;
  45:         _disableInitializers();
  46:         sale = Sale(0, 0, address(0), type(uint96).max, payable(0), type(uint96).max);
  47:     }


test/FixedPrice.t.sol:
  11  
  12:     function setUp() public virtual override {
  13:         super.setUp();
  14:         fixedSales = new FixedPriceFactory();
  15:         fixedSale = FixedPrice.Sale({
  16:             currentId: uint48(0),   // Same 
  17:             finalId: uint48(10),      // Not same
  18:             edition: address(edition), // Same
  19:             price: uint96(uint256(1 ether)),  // Not Same
  20:             saleReceiver: payable(address(69)), // Not Same
  21:             startTime: uint96(block.timestamp) // Not Same
  22:         });
  23:     }

```

### [N-03] Constant values such as a call to keccak256(), should used to immutable rather than constant

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand.

Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.


```js
4 results 2 files:

src/Escher.sol:
  11:     bytes32 public constant CREATOR_ROLE = keccak256("CREATOR_ROLE");
  13:     bytes32 public constant CURATOR_ROLE = keccak256("CURATOR_ROLE");

src/Escher721.sol:
  17:     bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");
  19:     bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```


### [N-04 ] For functions, follow Solidity standard naming conventions

```solidity
src/uris/Generative.sol:
  26  
  27:     function tokenToSeed(uint256 _tokenId) internal view returns (bytes32) {
```

**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

### [N-05] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last


### [N-06] Lack of event emission after critical `initialize()` function

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the `initialize()` function:

```solidity
1 result 1 file

src/Escher721.sol:
  32:     function initialize(
  33:         address _creator,
  34:         address _uri,
  35:         string memory _name,
  36:         string memory _symbol
  37:     ) public initializer {
  38:         __ERC721_init(_name, _symbol);
  39:         __AccessControl_init();
  40: 
  41:         tokenUriDelegate = _uri;
  42: 
  43:         _grantRole(DEFAULT_ADMIN_ROLE, _creator);
  44:         _grantRole(MINTER_ROLE, _creator);
  45:         _grantRole(URI_SETTER_ROLE, _creator);
  46:     }

```

### [N-07] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).
Consider adding a timelock to:

```solidity
3 results 3 files

src/minters/FixedPriceFactory.sol:
  47      // @audit Alice Bob , çok yüksek fee durumunda frontrunning yapılabilir
  48:     function setFeeReceiver(address payable fees) public onlyOwner {
  49:         feeReceiver = fees;
  50      }

src/minters/LPDAFactory.sol:
  45      /// @param fees the address to receive fees
  46:     function setFeeReceiver(address payable fees) public onlyOwner {
  47:         feeReceiver = fees;
  48      }

src/minters/OpenEditionFactory.sol:
  41      /// @param fees the address to receive fees
  42:     function setFeeReceiver(address payable fees) public onlyOwner {
  43:         feeReceiver = fees;

```
### [N-08] `Empty blocks` should be _removed_ or _Emit_ something

**Description:**
Code contains empty block

```solidity
1 result 1 file

src/Escher721.sol:
  25:     constructor() {}

```

**Recommendation:**
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.


### [N-09] NatSpec comments should be increased in contracts

**Context:**
All Contracts

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
NatSpec comments should be increased in contracts


### [N-10 ] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List: 
pragma ^0.8.17   (all contracts)

**Recommendation:**
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### [N-11] Loss of precision due to rounding

Although the loss of precision is minimal, this information can be specified in the Natspec comments and documentation.

```solidity
3 results 3 files

src/minters/FixedPrice.sol:
  111:         ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);

src/minters/LPDA.sol:
  85:             uint256 fee = totalSale / 20;
  86              ISaleFactory(factory).feeReceiver().transfer(fee);

src/minters/OpenEdition.sol:
  92:         ISaleFactory(factory).feeReceiver().transfer(address(this).balance / 20);

```

### [N-12] Add parameter to Event-Emit

Some event-emit description hasn’t parameter. Add to parameter  for front-end website or client app , they can has that something has happened on the blockchain.


```solidity
3 results  3 files

src/minters/FixedPriceFactory.sol:
  48:     function setFeeReceiver(address payable fees) public onlyOwner {
  49          feeReceiver = fees;

src/minters/LPDAFactory.sol:
  45      /// @param fees the address to receive fees
  46:     function setFeeReceiver(address payable fees) public onlyOwner {
  47          feeReceiver = fees;

src/minters/OpenEditionFactory.sol:
  41      /// @param fees the address to receive fees
  42:     function setFeeReceiver(address payable fees) public onlyOwner {
  43          feeReceiver = fees;

```
### [N-13] Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

```solidity

3 results  3 files

src/minters/FixedPriceFactory.sol:
  48:     function setFeeReceiver(address payable fees) public onlyOwner {
  49          feeReceiver = fees;

src/minters/LPDAFactory.sol:
  45      /// @param fees the address to receive fees
  46:     function setFeeReceiver(address payable fees) public onlyOwner {
  47          feeReceiver = fees;

src/minters/OpenEditionFactory.sol:
  41      /// @param fees the address to receive fees
  42:     function setFeeReceiver(address payable fees) public onlyOwner {
  43          feeReceiver = fees;

```

### [N-14] Lack of event emission after critical `initialize()` functions

To record the init parameters for off-chain monitoring and transparency reasons, please consider emitting an event after the `initialize()` functions:

```solidity
src/Escher721.sol:
  31      /// @param _symbol the symbol of this contract
  32:     function initialize(
  33:         address _creator,
  34:         address _uri,
  35:         string memory _name,
  36:         string memory _symbol
  37:     ) public initializer {
  38:         __ERC721_init(_name, _symbol);
  39:         __AccessControl_init();
  40: 
  41:         tokenUriDelegate = _uri;
  42: 
  43:         _grantRole(DEFAULT_ADMIN_ROLE, _creator);
  44:         _grantRole(MINTER_ROLE, _creator);
  45:         _grantRole(URI_SETTER_ROLE, _creator);
  46:     }

```

### [N-15] NatSpec is missing 

**Description:**
NatSpec is missing for the following functions , constructor and modifier:

**Context:**

```solidity
11 results 6 files

src/Escher721Factory.sol:
  12:     Escher public immutable escher;
  13:     address public immutable implementation;

  17:     constructor(address _escher) {

src/Escher.sol:
  31:     function supportsInterface(

src/Escher721.sol:
  84:     function supportsInterface(

src/minters/FixedPriceFactory.sol:
  12:     address payable public feeReceiver;
  13:     address public immutable implementation;

src/minters/LPDAFactory.sol:
  12:     address payable public feeReceiver;
  13:     address public immutable implementation;

src/minters/OpenEditionFactory.sol:
  12:     address payable public feeReceiver;
  13:     address public immutable implementation;

```

### [S-01] Mark visibility of initialize(...) functions as `external`

**Description:**
If someone wants to extend via inheritance, it might make more sense that the overridden initialize(...) function calls the internal {...}_init function, not the parent public initialize(...) function.

External instead of public would give more the sense of the initialize(...) functions to behave like a constructor (only called on deployment, so should only be called externally)

Security point of view, it might be safer so that it cannot be called internally by accident in the child contract 

It might cost a bit less gas to use external over public 


It is possible to override a function from external to public (= "opening it up") ✅
but it is not possible to override a function from public to external (= "narrow it down"). ❌

For above reasons you can change initialize(...) to external


https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750


### [S-02] Generate perfect code headers every time

**Description:**
I recommend using header for Solidity code layout and readability

https://github.com/transmissions11/headers

```js
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```