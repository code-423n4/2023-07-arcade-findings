## [G-01] `ARCDVestingVault.revokeGrant()`: `grant.latestVotingPower` to zero not needed in 

## Details and Recommendation
In `ARCDVestingVault.revokeGrant()`, the reassignment of `grant.latestVotingPower` to zero is not needed as this is already updated in `_syncVotingPower()`.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L184
```solidity
function revokeGrant(address who) external virtual onlyManager {
    // load the grant
    ARCDVestingVaultStorage.Grant storage grant = _grants()[who];

    // if the grant has already been removed or no grant available, revert
    if (grant.allocation == 0) revert AVV_NoGrantSet();

    // get the amount of withdrawable tokens
    uint256 withdrawable = _getWithdrawableAmount(grant);
    grant.withdrawn += uint128(withdrawable);
    token.safeTransfer(who, withdrawable);

    // transfer the remaining tokens to the vesting manager
    uint256 remaining = grant.allocation - grant.withdrawn;
    grant.withdrawn += uint128(remaining);
    token.safeTransfer(msg.sender, remaining);

    // update the delegatee's voting power
    _syncVotingPower(who, grant);

    // delete the grant
    grant.allocation = 0;
    grant.cliffAmount = 0;
    grant.withdrawn = 0;
    grant.created = 0;
    grant.expiration = 0;
    grant.cliff = 0;
--  grant.latestVotingPower = 0;
    grant.delegatee = address(0);
}
```

## [G-02] `ARCDVestingVault.revokeGrant()`: Check zero withdrawals 

## Details and Recommendation
If `withdrawable` is assigned to zero via `_getWithdrawableAmount()`, consider reverting the function to save gas for manager calling the `revokeGrant()` function.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L165

```solidity
function revokeGrant(address who) external virtual onlyManager {
    // load the grant
    ARCDVestingVaultStorage.Grant storage grant = _grants()[who];

    // if the grant has already been removed or no grant available, revert
    if (grant.allocation == 0) revert AVV_NoGrantSet();

    // get the amount of withdrawable tokens
    uint256 withdrawable = _getWithdrawableAmount(grant);
++  if (withdrawable == 0) revert CannotRevoke();
    grant.withdrawn += uint128(withdrawable);
    token.safeTransfer(who, withdrawable);

    // transfer the remaining tokens to the vesting manager
    uint256 remaining = grant.allocation - grant.withdrawn;
    grant.withdrawn += uint128(remaining);
    token.safeTransfer(msg.sender, remaining);

    // update the delegatee's voting power
    _syncVotingPower(who, grant);

    // delete the grant
    grant.allocation = 0;
    grant.cliffAmount = 0;
    grant.withdrawn = 0;
    grant.created = 0;
    grant.expiration = 0;
    grant.cliff = 0;
    grant.latestVotingPower = 0;
    grant.delegatee = address(0);
}
```