# Table of Contents

- [Table of Contents](#table-of-contents)
- [Summary](#summary)
	- [Files Summary](#files-summary)
	- [Files Details](#files-details)
	- [Issue Summary](#issue-summary)
- [High Issues](#high-issues)
	- [H-1: `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`](#h-1-abiencodepacked-should-not-be-used-with-dynamic-types-when-passing-the-result-to-a-hash-function-such-as-keccak256)
- [Low Issues](#low-issues)
	- [L-1: Centralization Risk for trusted owners](#l-1-centralization-risk-for-trusted-owners)
	- [L-2: Solidity pragma should be specific, not wide](#l-2-solidity-pragma-should-be-specific-not-wide)
	- [L-3: Missing checks for `address(0)` when assigning values to address state variables](#l-3-missing-checks-for-address0-when-assigning-values-to-address-state-variables)
	- [L-4: `public` functions not used internally could be marked `external`](#l-4-public-functions-not-used-internally-could-be-marked-external)
	- [L-5: Define and use `constant` variables instead of using literals](#l-5-define-and-use-constant-variables-instead-of-using-literals)
	- [L-6: Event is missing `indexed` fields](#l-6-event-is-missing-indexed-fields)
	- [L-7: Loop contains `require`/`revert` statements](#l-7-loop-contains-requirerevert-statements)


# Summary

## Files Summary

| Key | Value |
| --- | --- |
| .sol Files | 2 |
| Total nSLOC | 138 |


## Files Details

| Filepath | nSLOC |
| --- | --- |
| src/PuppyRaffle.sol | 138 |
| **Total** | **138** |


## Issue Summary

| Category | No. of Issues |
| --- | --- |
| High | 1 |
| Low | 7 |


# High Issues

## H-1: `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`

Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). Unless there is a compelling reason, `abi.encode` should be preferred. If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead.

- Found in src/PuppyRaffle.sol [Line: 271](src/PuppyRaffle.sol#L271)

	```solidity
	            abi.encodePacked(
	```

- Found in src/PuppyRaffle.sol [Line: 275](src/PuppyRaffle.sol#L275)

	```solidity
	                        abi.encodePacked(
	```



# Low Issues

## L-1: Centralization Risk for trusted owners

Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

- Found in src/PuppyRaffle.sol [Line: 21](src/PuppyRaffle.sol#L21)

	```solidity
	contract PuppyRaffle is ERC721, Ownable {
	```

- Found in src/PuppyRaffle.sol [Line: 234](src/PuppyRaffle.sol#L234)

	```solidity
	    function changeFeeAddress(address newFeeAddress) external onlyOwner {
	```



## L-2: Solidity pragma should be specific, not wide

Consider using a specific version of Solidity in your contracts instead of a wide version. For example, instead of `pragma solidity ^0.8.0;`, use `pragma solidity 0.8.0;`

- Found in src/PuppyRaffle.sol [Line: 2](src/PuppyRaffle.sol#L2)

	```solidity
	pragma solidity ^0.7.6;
	```



## L-3: Missing checks for `address(0)` when assigning values to address state variables

Check for `address(0)` when assigning values to address state variables.

- Found in src/PuppyRaffle.sol [Line: 76](src/PuppyRaffle.sol#L76)

	```solidity
	        feeAddress = _feeAddress;
	```

- Found in src/PuppyRaffle.sol [Line: 235](src/PuppyRaffle.sol#L235)

	```solidity
	        feeAddress = newFeeAddress;
	```



## L-4: `public` functions not used internally could be marked `external`

Instead of marking a function as `public`, consider marking it as `external` if it is not used internally.

- Found in script/DeployPuppyRaffle.sol [Line: 12](script/DeployPuppyRaffle.sol#L12)

	```solidity
	    function run() public {
	```

- Found in src/PuppyRaffle.sol [Line: 93](src/PuppyRaffle.sol#L93)

	```solidity
	    function enterRaffle(address[] memory newPlayers) public payable {
	```

- Found in src/PuppyRaffle.sol [Line: 117](src/PuppyRaffle.sol#L117)

	```solidity
	    function refund(uint256 playerIndex) public {
	```

- Found in src/PuppyRaffle.sol [Line: 263](src/PuppyRaffle.sol#L263)

	```solidity
	    function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
	```



## L-5: Define and use `constant` variables instead of using literals

If the same constant literal value is used multiple times, create a constant state variable and reference it throughout the contract.

- Found in src/PuppyRaffle.sol [Line: 175](src/PuppyRaffle.sol#L175)

	```solidity
	        uint256 prizePool = (totalAmountCollected * 80) / 100;
	```

- Found in src/PuppyRaffle.sol [Line: 176](src/PuppyRaffle.sol#L176)

	```solidity
	        uint256 fee = (totalAmountCollected * 20) / 100;
	```

- Found in src/PuppyRaffle.sol [Line: 192](src/PuppyRaffle.sol#L192)

	```solidity
	        uint256 rarity = uint256(keccak256(abi.encodePacked(msg.sender, block.difficulty))) % 100;
	```



## L-6: Event is missing `indexed` fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

- Found in src/PuppyRaffle.sol [Line: 64](src/PuppyRaffle.sol#L64)

	```solidity
	    event RaffleEnter(address[] newPlayers);
	```

- Found in src/PuppyRaffle.sol [Line: 65](src/PuppyRaffle.sol#L65)

	```solidity
	    event RaffleRefunded(address player);
	```

- Found in src/PuppyRaffle.sol [Line: 66](src/PuppyRaffle.sol#L66)

	```solidity
	    event FeeAddressChanged(address newFeeAddress);
	```



## L-7: Loop contains `require`/`revert` statements

Avoid `require` / `revert` statements in a loop because a single bad item can cause the whole transaction to fail. It's better to forgive on fail and return failed elements post processing of the loop

- Found in src/PuppyRaffle.sol [Line: 105](src/PuppyRaffle.sol#L105)

	```solidity
	            for (uint256 j = i + 1; j < players.length; j++) { //then we loop through the players array again j
	```



