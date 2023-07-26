## G-01 `startTime` in `ImmutableVestingVault.sol` is not used for anything 

When calling `addGrantAndDelegate()` in `ImmutableVestingVault.sol` a grand is created and among others things the `startTime` of the grand is set (`grant.created`). This start time is not used for anything since the `cliff` determines when the `cliffAmount` is unlocked and the `cliff` is a fixed block number and not a time periode after the `startTime`. This means the variable `grant.created`) can be removed from the grant struck safely and gas would be saved when initiating a grand.


## G-02 Fail early in case `delegatee` is 0 when calling `delegate()` in `NFTBoostVault.sol`

When calling `delegate()` in `NFTBoostVault.sol` the `registration` is called and the delegate is set was well as the voting power of the old an the new delegate is changed. If the `registration` of the user is not initiated yet the whole function executes and wastes gas since there is no voting power to redistribute. Consider adding a check after the `registration` is called that checks if the delegate is 0. If so revert the function. 

### Code
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L182-L212 


## G-03 Save gas by not setting `latestVotingPower` to 0 in `withdraw()` in `NFTBoostVaul.sol`

When withdrawing all the tokens, at the end of the function the `latestVotingPower` of the user is set to 0. This is not necessary since it is already set to 0 when calling ` _syncVotingPower ` a couple of lines before since `amount- withdrawn` equal 0.

### Code 

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L244-L253

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L579-L599




