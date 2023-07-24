## desc
see this code, I think the logic here is redundant.
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L237C1-L252C54

This code, I think, is more concise.
```
uint256 withdrawable = _getWithdrawableAmount(grant);
if (amount > withdrawable) revert AVV_InsufficientBalance(withdrawable);

grant.withdrawn += uint128(amount);

// update the user's voting power
_syncVotingPower(msg.sender, grant);

// transfer the available amount
token.safeTransfer(msg.sender, amount);
```

I found similar logic in this git, and his implementation matches mine
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L234-L258



