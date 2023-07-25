### Low Risk Issues
| Count | Title | 
|:--:|:-------|:--:|
| [L-01] | `ArcadeTreasury._spend()/_approve()`: Can spend tokens where thresholds has not been set | 
| [L-02] | `ARCDVestingVault.revokeGrant()`: If claiming and revoking grants occurs in same block, manager maybe DoS from revoking grants| 


| Total Low Risk Issues | 2 |
|:--:|:--:|

### Non-Critical Issues
| Count | Title | 
|:--:|:-------|:--:|
| [N-01] | Add zero address checks for `delegate()` | 
| [N-02] | `NFTBoostVault.setMultiplier()`: Add minimum multiplier check in `setMultiplier`| 

| Total Non-Critical Issues | 2 |
|:--:|:--:|

### Refactor Issues
| Count | Title | 
|:--:|:-------|:--:|
| [R-01] | `ARCDVestingVault.revokeGrant()`: `grant.latestVotingPower` to zero not needed in | 
| [R-02] | `ARCDVestingVault.revokeGrant()`: Check zero withdrawals | 

| Total Refactor Issues | 2 |
|:--:|:--:|

## [L-01] `ArcadeTreasury._spend()/_approve()`: Proposals can spend tokens where thresholds has not been set

## Impact
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L358

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L384

Proposals can spend Arcade tokens from token addresses that has yet to been set, potentially by-passing the intended spending limits. Consider adding a check in `_spend()` and `_approve()` to prevent this from occuring.

## Recommendation
```solidity
function _spend(address token, uint256 amount, address destination, uint256 limit) internal {
++  if (limit == 0) revert T_ThresholdNotSet;

    // check that after processing this we will not have spent more than the block limit
    uint256 spentThisBlock = blockExpenditure[block.number];
    if (amount + spentThisBlock > limit) revert T_BlockSpendLimit();
    blockExpenditure[block.number] = amount + spentThisBlock;

    // transfer tokens
    if (address(token) == ETH_CONSTANT) {
        // will out-of-gas revert if recipient is a contract with logic inside receive()
        payable(destination).transfer(amount);
    } else {
        IERC20(token).safeTransfer(destination, amount);
    }

    emit TreasuryTransfer(token, destination, amount);
}
```

```solidity
function _approve(address token, address spender, uint256 amount, uint256 limit) internal {
++  if (limit == 0) revert T_ThresholdNotSet;
    // check that after processing this we will not have spent more than the block limit
    uint256 spentThisBlock = blockExpenditure[block.number];
    if (amount + spentThisBlock > limit) revert T_BlockSpendLimit();
    blockExpenditure[block.number] = amount + spentThisBlock;

    // approve tokens
    IERC20(token).approve(spender, amount);

    emit TreasuryApproval(token, spender, amount);
}
```

## [L-02] `ARCDVestingVault.revokeGrant()`: If claiming and revoking grants occurs in same block, manager maybe DoS from revoking grants

## Impact
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L157-L186

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L228-L253

If grant is revoked at the same block where claiming of grants is available, then revoking of grant will be side stepped and grant owners will receive arcade tokens for voting and proposing (delegatees if assigned).


## Recommendation
Add a additional delay (1 block) before allowing claiming of grants.

## [NC-01] Add zero address checks for `delegate()`

## Impact
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L260-L282

Voters can accidentally delegate voting power to zero-addresses causing temporary losing of voting power

## Recommendation
```solidity
function delegate(address to) external {
    ARCDVestingVaultStorage.Grant storage grant = _grants()[msg.sender];
++  if (to == address(0)) revert AVV_AlreadyDelegated();
    if (to == grant.delegatee) revert AVV_AlreadyDelegated();

    History.HistoricalBalances memory votingPower = _votingPower();
    uint256 oldDelegateeVotes = votingPower.loadTop(grant.delegatee);

    // Remove old delegatee's voting power and emit event
    votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);
    emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));

    // Note - It is important that this is loaded here and not before the previous state change because if
    // to == grant.delegatee and re-delegation was allowed we could be working with out of date state.
    uint256 newDelegateeVotes = votingPower.loadTop(to);

    // add voting power to the target delegatee and emit event
    votingPower.push(to, newDelegateeVotes + grant.latestVotingPower);

    // update grant delgatee info
    grant.delegatee = to;

    emit VoteChange(to, msg.sender, int256(uint256(grant.latestVotingPower)));
}
```
Add zero address checks for `delegate()` functions in `ARCDVestingVault` similar to that in `NFTBoostVault`.

## [NC-02] `NFTBoostVault.setMultiplier()`: Add minimum multiplier check in `setMultiplier`

## Impact
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L363-L371

The minimum multiplier is 1e13 (100%). Consider adding a check to not allow ADMIN_ROLE setting voter's multiplier below 100% that can potentially unfairly reduce voting power

## Recommendation
```solidity
function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {
--  if (multiplierValue > MAX_MULTIPLIER) revert NBV_MultiplierLimit();
++  if (multiplierValue > MAX_MULTIPLIER && multiplierValue < 1e3) revert NBV_MultiplierLimit();

    NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];
    // set multiplier value
    multiplierData.multiplier = multiplierValue;

    emit MultiplierSet(tokenAddress, tokenId, multiplierValue);
}
```

## [R-01] `ARCDVestingVault.revokeGrant()`: `grant.latestVotingPower` to zero not needed in 

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

## [R-02] `ARCDVestingVault.revokeGrant()`: Check zero withdrawals 

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
