## Usage of `selfdestruct`
After [EIP-4758](https://eips.ethereum.org/EIPS/eip-4758), the `SELFDESTRUCT` op code will no longer be available. Currently, the project uses `selfdestruct` in `FixedPrice` and `OpenEdition` to send the funds (and destroy the contract) after an auction has ended. The sending of the funds should still work (even after EIP-4758), but it may still be desirable to not rely on an opcode that will be removed in the future (and where the exact behavior afterwards is not known yet).
## `transfer` used in multiple places
In multiple places, `transfer` is used for sending ETH. Because the function only provides a 2300 gas stipend, this may fail with smart contract wallets that contain some logic in its `receive` function.
## Fee Receiver can grift sales
When a sale is the last lase within `FixedPrice` and `LPDA`, the fees are transfered to the fee receiver (i.e., push transfer). The fee receiver could revert on purpose to grift a sale, e.g. to avoid that a certain user buys a token or to buy the last one himself and sell it OTC for a higher price. This may be undesirable, so using a pull-based pattern may be more appropriate.
## Fee Receiver can be set to `address(0)`, resulting in burned fees
In the three factories, `setFeeReceiver` can be used to set the fee receiver to `address(0)`. In such cases, the transfer of the fees will still be initiated, but it will result in burning the ETH. Consider disallowing setting the address to 0 or only initiating a transfer of fees when it is non-zero.
## Wrong comments in FixedPrice.buy and LPDA.buy
In the two functions, the comment above the function says: "/// @notice buy from a fixed price sale after the sale starts".
This is wrong (and probably a copy-past error from `FixedPrice`).
## Generative.tokenToSeed completely determined after setting the seed base
After the seed base is set, everyone can calculate the result of `tokenToSeed` for all token IDs. Because the function is not used within the codebase, the impact of that is not clear to me. However, if it should be used for randomness (e.g., NFT reveals), this would be very bad because users could already determine their NFT before the on-chain reveal. For that, Chainlink VRF would be a good alternative.