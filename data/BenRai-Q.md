Low!!!  

## L-01 Because grands are not revocable in `ARCDVestingVault.sol` tokens can be stuck in the contract for ever and their voting power can be lost

If the private key of an address that received a grand gets lost it will never be able to claim the granted tokens or execute the voting power connected to the tokens and the tokens as well as the token voting power will be lost.

### Code
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ImmutableVestingVault.sol#L40-L43

### Recomendstion
Think about adding a time period that determines how long after the vesting time is over the tokens must be claimed. Once this period is over make the grand revocable to be able to “save” the tokens and give them e.g back to the treasury.

## L-02 There is no option to revoke an approval given by the GSC in `ArcadeTreasury.sol`
Once the GSC approves an address to spend some tokens, there is no way to revoke the approval again. Consider allowing a 0 approval to be able to revoke an approval given by the GSC and also increasing the `gscAllowance` of the token when reducing the given approval. A check of the current amount of the approval would be necessary for that.  

### Code
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L189-L201

## L-03 Allowing `timelock` in `BaseVotingVault` to change the `timelock address` in one step can be problematic

Consider making the process of changing the `timelock address` by the `timelock` a 2 step process like used in `Ownable2Step`. This would prevent `timelock` to change the `timelock address` to a wrong address and rendering the functions only usable by `timelock` unusable 

### Code
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L68-L72

## L-04 Allowing ` minter ` in `ArcadeToken.sol` to change the ` minter address` in one step can be problematic

Consider making the process of changing the `minter address` by the `minter` a 2 step process like used in `Ownable2Step`. This would prevent `minter` to change the `minter address` to a wrong address, that can result in the fact that no new tokens can be minted. 

### Code
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132-L137


## L-05 Consider adding a check if the multiplier is smaller than 1 when calling `setMultiplier` in `NFTBoostVault.sol`

Consider adding a check if the multiplier is smaller than 1 when calling `setMultiplier` in `NFTBoostVault.sol` to prevent the locked tokens to have less than 1 vote per token
### Code 
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L363-L371
