## Finding 1:
In `Echer721Factory` the `creteContract` function is used to create new `Echer721` clone. In the process it also checked if `msg.sender` has creator role. To do that it first gets the role identifier by calling `echer.CREATOR_ROLE()`. However, `CREATOR_ROLE` is a constant so you could just use `keccak256("CREATOR_ROLE")` and avoid making an external call which uses a lot more gas.

### Recommendation:
Change this line (32) in `Escher721Factory.sol`
```
require(escher.hasRole(escher.CREATOR_ROLE(), msg.sender), "NOT AUTHORIZED");
```
to this
```
require(escher.hasRole(keccak256("MINTER_ROLE"), msg.sender), "NOT AUTHORIZED");
```


## Finding 2:
In all kinds of sales, there are 3 events: `Start`, `End` and `Buy`. All these events include `Sale _saleInfo`. Furthermore there is an event emitted in factories when sale is created that also includes sale information. However, including all the information about a sale in every event is unnecessary. In `FixedPrice` for example, the only variable that changes through the process of a sale is `currentId`. This means that all the other variables are the same as in the past events and therefore can be read from those. Also, `Buy` event includes `_amount` of bought NFTs, which can be used together with data from previous events to calculate what the `currentId` is.

### Recommendation:
Consider changing events like so:
In factories remove `saleInfo`.
In sales only include `saleInfo` in `Start` event and remove it from others.

