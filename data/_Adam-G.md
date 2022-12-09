### [G01] x = x + y is Cheaper than x += y

**Description:**
Based on test in remix you can save ~1,007 gas on deployment and ~15 gas on execution cost if you use x = x + y over x += y (Is only true for Storage Variables).

```
contract Test {
	uint256 x = 1;
	function test() external {
		x += 3; 
		(Deployment Cost: 153,124, Execution Cost: 30,369)
		vs
		x = x + 1;
		(Deployment Cost: 152,117, Execution Cost: 30,354)
	}

}
```

**LOC:**
[LPDA.sol#L66](https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/LPDA.sol#L66) 
[LPDA.sol#L70](https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/LPDA.sol#L70) 
[LPDA.sol#L71](https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/LPDA.sol#L71) 

**Recommendation:**
Change line 66 to: amountSold = amountSold + amount;
Change line 70 to: receipts[msg.sender].amount = receipts[msg.sender].amount + amount;
Change line 71 to: receipts[msg.sender].balance = receipts[msg.sender].balance + uint80(msg.value);


### [G02] Unchecked Increments in Loops 

**Description:**
When incrementing i in for loops there is no chance of overflow so unchecked can be used to save gas. I ran a simple test in remix and found deployment savings of 31,653 gas and on each function call saved ~141 gas per iteration. 

``` 
contract Test { 
	function loopTest() external { 
		for (uint256 i; i < 1; ++i) { 
			Deployment Cost: 125,637, Cost on function call: 24,601 
			
		vs 
			
		for (uint256 i; i < 1; ) { 
			// for loop body 
			unchecked { ++i } 
			Deployment Cost: 93,984, Cost on function call: 24,460 
		} 
	} 
} 
```

**LOC:**
[FixedPrice.sol#L65-L67](https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/FixedPrice.sol#L65-L67) 
[LPDA.sol#L73-L75](https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/LPDA.sol#L73-L75) 
[OpenEdition.sol#L66-L68](https://github.com/code-423n4/2022-12-escher/blob/b60ce318436aaccd00d0701335049726a6f792ff/src/minters/OpenEdition.sol#L66-L68) 

**Recommendation:**
Move the loop increments to the end of the loops body and wrap in an unchecked block.