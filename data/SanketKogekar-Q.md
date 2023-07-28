
- The `setThreshold()` function checks for the case: 

```solidity
if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small) {
    revert T_ThresholdsNotAscending();
}
```

But devs should note that in case all threshold values are equal, then it accept and never reverts:

thresholds.small == thresholds.medium == thresholds.large 

So `thresholds.small` could be equal to the `thresholds.large`

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L276-L278

- The `receive()` function is public and accepts Ether, but it does not have any validation or handling logic. This could lead to unintended Ether transfers to the contract, which might get trapped inside the contract and be irretrievable.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L397

- The `address(0)` is validated at beginning of every function in such way:

`if (user == address(0)) revert AVV_ZeroAddress("manager");`

except the `to` param is not validated on `delegate()` and power could be delegated to `address(0)` by any user.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L260

- The `mint()` function misses the `amount > 0` check. The function works correct without loss of funds or incorrect accounting, but most protocols prefer to revert (and so do other functions on same contracts)

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L98-L120

- The contract fails to use OpenZeppelin's or equivalent `safeApprove()`, `safeTransfer()` and `safeMint()` methods in certain functions.

- The `publishRoots()` does not ensure that the provided `tokenId` exists and that the claim data is valid when calling the `publishRoots()` function.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L140-L156

- The `_verifyClaim()` does not check `claimRoots` mapping value, which can happen when token with certain `tokenId` is never minted.

When: `claimRoots[tokenId] == bytes32(0)`

If `rewardsRoot` is uninitialized, then the claim cannot be verified. This would allow an attacker to mint badges that they are not entitled to.

If this code snippet is added:

```solidity
if (rewardsRoot == bytes32(0)) {
    return false;
}
```

It will check if the `rewardsRoot` is initialized before attempting to verify the claim. If the `rewardsRoot` is not initialized, then the function will return false. This will prevent an attacker from minting badges that they are not entitled to.

- When using `mint()` the excess fee (`msg.value` which is sent by user to match the `mintPrice`) is not returned to user, and will be stuck in contract. Note that `withdrawFees()` function will transfer all the contract's balance to `recipient` when called by `BADGE_MANAGER_ROLE` but user will lose his tokens.

Mitigation: Send rest of the `msg.value` to the user (after deducting `mintPrice` which is the 'fee' as descibed by the protocol)

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L109

