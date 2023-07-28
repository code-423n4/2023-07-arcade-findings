# Summary

| Id   | Title                                           |
|------|-------------------------------------------------|
| L-01 | Grant `startTime` can be retroactive            |
| L-02 | Missing events in `spending/approval` functions |
| L-03 | Storage `initialized` is never used             |
| L-04 | `DAY_IN_BLOCKS` is too small after PoS          |
| L-05 | NFT URI can be changed anytime                  |
| L-06 | State variable shadowing                        |


## [L-01] Grant `startTime` can be retroactive

The grant `startTime` should be able to be set only at the current block or in the future:

```solidity
* @param startTime                  Optionally set a start time in the future. If set to zero
*                                   then the start time will be made the block tx is in.
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L84-L85

But there aren't any checks to ensure that this can't happen. As such, it's possible to add a retroactive grant with a `startTime` in the past, that can be vested instantly:

```solidity
// if no custom start time is needed we use this block.
if (startTime == 0) {
    startTime = uint128(block.number);
}
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L104-L107

## [L-02] Missing events in `spending/approval` functions

`ArcadeTreasury` has different functions to allow spending, but the event emitted is shared, as it's emitted in `_spend`:

```solidity
function smallSpend(
    address token,
    uint256 amount,
    address destination
) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
    if (destination == address(0)) revert T_ZeroAddress("destination");
    if (amount == 0) revert T_ZeroAmount();

    _spend(token, amount, destination, spendThresholds[token].small);
}

function mediumSpend(
    address token,
    uint256 amount,
    address destination
) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
    if (destination == address(0)) revert T_ZeroAddress("destination");
    if (amount == 0) revert T_ZeroAmount();

    _spend(token, amount, destination, spendThresholds[token].medium);
}
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L130-L139
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L149-L158

This means that it's difficult to track if anyone called either `smallSpend/mediumSpend/largeSpend`, as the only data available is the amount spent, which is variable.

Consider adding a separate event for every spending function that fully describes what is happening.

## [L-03] Storage `initialized` is never used

The `initialized` slot is assigned, but there isn't any getter function for this value, unlike others:

```solidity
Storage.set(Storage.uint256Ptr("initialized"), 1);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L91

## [L-04] `DAY_IN_BLOCKS` is too small after PoS

`ArcadeGSCCCoreVoting` inherits a `DAY_IN_BLOCKS` variable which is not up to date. The correct value should be `7148` (which is ~12% bigger) because after the PoS merge the network consistently produces a block every ~12 seconds:

```solidity
// Assumes avg block time of 13.3 seconds. May be longer or shorter due
// to ice ages or short term changes in hash power.
uint256 public constant DAY_IN_BLOCKS = 6496;
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/external/council/CoreVoting.sol#L13-L15

For this reason, lock durations will take less time than expected before they are unlocked.

## [L-05] NFT URI can be changed anytime

The owner is able to change the `BadgeDescriptor` URI anytime, and it can't be locked:

```solidity
function setBaseURI(string memory newBaseURI) external onlyOwner {
    baseURI = newBaseURI;

    emit SetBaseURI(msg.sender, newBaseURI);
}
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L57-L61

## [L-06] State variable shadowing

Some state variables are shadowed by local variables/function parameters, this might result in the use of the wrong variable, in some scenarios. Consider renaming these variables with a different name.

```solidity
File: contracts/ArcadeGSCCoreVoting.sol

// @audit baseQuorum inherited from CoreVoting
28: 		        uint256 baseQuorum,

// @audit minProposalPower inherited from CoreVoting
29: 		        uint256 minProposalPower,
```
https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/ArcadeGSCCoreVoting.sol#L28-L29


```solidity
File: contracts/ArcadeGSCVault.sol

// @audit coreVoting inherited from GSCVault
36: 		        ICoreVoting coreVoting,

// @audit votingPowerBound inherited from GSCVault
37: 		        uint256 votingPowerBound,
```
https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/ArcadeGSCVault.sol#L36-L37

```solidity
File: contracts/NFTBoostVault.sol

// @audit token inherited from BaseVotingVault
83: 		        IERC20 token,

// @audit staleBlockLag inherited from BaseVotingVault
84: 		        uint256 staleBlockLag,
```
https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/NFTBoostVault.sol#L83-L84