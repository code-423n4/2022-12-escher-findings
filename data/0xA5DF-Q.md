
## Attacker can buy zero amount in order to spam the contract with false events
They `buy()` function doesn't require the amount to purchase to be greater than zero, meaning a user can run the function with zero amount, which would spam the contract with `Buy` events without actually buying anything.

## It's not possible to mint token #0 with the sales contracts
Code [src/minters/LPDA.sol#L73](https://github.com/code-423n4/2022-12-escher/blob/11ca988e39effe6d316e996c27a876c92e82b4da/src/minters/LPDA.sol#L73)

It's not possible to mint the #0 token, since the sales contract will start minting the `temp.currentId + 1` token, and since `currentId` is an unsigned int it can't be lower then 0. Meaning the minimum token ID that can be minted is 1.

### Mitigation
Switch the currentID to be the next token ID to be minted, rather than the last token minted, and update the code accordingly.


## Unsafe cast of `msg.value` can cause loss of ETH in a rare case
Code: [src/minters/LPDA.sol#L71](https://github.com/code-423n4/2022-12-escher/blob/11ca988e39effe6d316e996c27a876c92e82b4da/src/minters/LPDA.sol#L71)

The `LPDA.buy()` function unsafely casts the `msg.value` to `uint80`, in case that `msg.value` is greater than `2^80` it'll reduce the amount to `msg.value % 2^80`, and the difference will be locked in the contract.
This is not very practical since it requires to send ~1.2M ETH, but it's still better to do a safe cast (i.e. check that `msg.value <= type(uint80).max`) just in case.

## Unsafe cast of `_amount` at `buy()`
The `buy()` function gets a `uint256` parameter named `_amount` and unsafely casts it to uint24/uint48.

As for the OpenEdition and LPDA contracts, this would have only a 'cosmetic' effect (user who'd look at the tx would see that the `_amount` parameter sent is much greater than the actual amount purchased).

As for the Fixed Price contract, this would also cause a loss of value, since the `_amount` is casted only for calculating the `newID` but not for the `msg.value` check.
This can lead to a rare case where a user sets `_amount` to `x + 2^48` and send ETH accordingly, but get only x tokens in return.

## Open Edition price is limited to 4.7K ETH
The `Sale.price` variable of the `OpenEdition` contract is a `uint72`, meaning it's max value is ~4.7K ETH.
For most sales that would be enough, but in the history of NFT there have been recorded higher prices.
The size of the variable can be increased without taking up any additional storage slots, by moving the `price` variable to the 3rd slot of the `Sale` struct:

```solidity
    struct Sale {
        // slot 1
        uint48 currentId;
        address edition;
        // slot 2
        uint96 startTime;
        address payable saleReceiver;
        // slot 3
        uint96 endTime;
        uint96 price;
    }
```

Also the LDPA price variable can be increased to `uint84` instead of `uint80` without taking up any additional slots.

## `Generative.tokenToSeed()` is internal instead of public

`Generative.tokenToSeed()` function is defined as internal, but has no internal usage.
I understood from the devs that it's supposed to  serve external contract in generating metadata from a seed. Therefore it needs to be changed to public/external in order to be accessible.

## Any user can create a sale with a cloned `Escher721` contract
The sale contract factories only check that the creator has the admin role over the `_sale.edition` contract.
But any user can simply clone the code of `Escher721`, deploy his own version without having the Creator role and then start a sale with it.
This might confuse users to thing that the created sale contract is from a legitimate seller approved by the protocol.

### Mitigation
The cheapest/easiest solution would be to also check that `msg.sender` has the Creator role.
A more thorough solution would be to handle a registry of deployed contract at `Escher721Factory`, and at the sale factories check that the `edition` contract is registered there.