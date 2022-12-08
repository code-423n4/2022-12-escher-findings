# Gas Report

## 1. Use `unchecked { ++x; }` within for loops, as the index cannot overflow

```
function buy(...) external payable {

    //...

    for (uint256 x = temp.currentId + 1; x <= newId;) {
        nft.mint(msg.sender, x);
        unchecked{ ++x; }
    }

    //...
}
```

1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/FixedPrice.sol#L65
2. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L73
3. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/OpenEdition.sol#L66

- Deployment saving: 45 gas
- Gas saving per iteration: 82 gas

## 2. Replace `x += y` with `x = x + y`

1. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L66
2. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L70
3. https://github.com/code-423n4/2022-12-escher/blob/main/src/minters/LPDA.sol#L71

- Deployment saving: 10 gas
- Gas saving per call: 210 gas
