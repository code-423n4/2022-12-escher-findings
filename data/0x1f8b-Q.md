- [Low](#low)
    - [**1. Improve tokenURI logic**](#1-improve-tokenuri-logic)
    - [**2. Front running in initialize method**](#2-front-running-in-initialize-method)
    - [**3. Owner can steal the fees**](#3-owner-can-steal-the-fees)
    - [**4. Contracts without GAP can lead a upgrade disaster**](#4-contracts-without-gap-can-lead-a-upgrade-disaster)
    - [**5. Integer overflow by unsafe casting**](#5-integer-overflow-by-unsafe-casting)
- [Non critical](#non-critical)
    - [**6. Use abstract for base contracts**](#6-use-abstract-for-base-contracts)

# Low

## **1. Improve `tokenURI` logic**

The `tokenURI` method does not check that `_tokenId` exists, so it might return uris for invalid tokens. This behavior should be copied from the logic of [Open Zeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/3dac7bbed7b4c0dbf504180c33e8ed8e350b93eb/contracts/token/ERC721/ERC721.sol#L94) where they check if the token was minted before returning a valid result.

**Affected source code:**

- [Unique.sol:10](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Unique.sol#L10)
- [Base.sol:15](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Base.sol#L15)

## **2. Front running in `initialize` method**

The methods that inherit the `Base` contract have a method that is susceptible to front-running attacks, which allows an attacker to obtain ownership and take advantage of the contract as soon as it is deployed, it is convenient to protect this type of initializations making sure that only a specific sender can call said method.

In the case of the 'Generative' contract, the attacker can call `setScriptPiece` with arbitrary data and exploit or combine other attacks.

**Affected source code:**

- [Base.sol:18](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Base.sol#L18)
- [Unique.sol:7](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Unique.sol#L7)
- [Generative.sol:6](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L6)

## **3. Owner can steal the fees**

If the owner calls `cancel`, it will not pass any fees to `feeReceiver`, so it is possible for the owner to avoid this commitment to the fee manager. All this fees will be transfered to `saleReceiver` instead of the `feeReceiver` as was agreed during the deployment.

**Affected source code:**

- [OpenEdition.sol:75](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L75)

## **4. Contracts without GAP can lead a upgrade disaster**

Some contracts do not implement good upgradeable logic.

Upgrading a contract will need a `__gap` storage in order to avoid storage collisions.

Proxied contracts MUST include an empty reserved space in storage, usually named `__gap`, in all the inherited contracts.

For example, the contract `Governed` or `GraphUpgradeable` are inherited by upgradeable contracts, but these contracts don't have a `__gap` storage, so if a new variable is created it could lead to storage collisions that will produce a contract disaster, including loss of funds.

**Reference:**
- https://docs.openzeppelin.com/contracts/3.x/upgradeable#storage_gaps
- https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/2cb8996b777060e658e2b8c9b1630313aedb04c0/contracts/token/ERC20/ERC20Upgradeable.sol#L394

**Affected source code:**

- [Unique.sol:7](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Unique.sol#L7)
- [Base.sol:7](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Base.sol#L7)
- [Generative.sol:7](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L7)
- [OpenEdition.sol:10](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L10)

**Recommended Mitigation Steps:**

- Add `uint256[50] private __gap;` to all upgradable contracts.

## **5. Integer overflow by unsafe casting**

Keep in mind that the version of solidity used, despite being greater than `0.8`, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

**Recommendation:**

Use a [safeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) from Open Zeppelin.

```javascript
uint80 price = uint80(getPrice()) * r.amount;
```

**Affected source code:**

- [FixedPrice.sol:62](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L62)
- [LPDA.sol:59](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L59)
- [LPDA.sol:71](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L71)
- [LPDA.sol:82](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L82)
- [LPDA.sol:101](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L101)
- [OpenEdition.sol:58](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L58)

---

# Non critical

## **6. Use `abstract` for base contracts**

`abstract` contracts are contracts that have at least one function without its implementation. **An instance of an abstract cannot be created.**

**Reference:**

- https://docs.soliditylang.org/en/v0.6.2/contracts.html#abstract-contracts

**Affected source code:**

- [Unique.sol:7](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Unique.sol#L7)
- [Base.sol:7](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Base.sol#L7)
- [Generative.sol:7](https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/uris/Generative.sol#L7)
