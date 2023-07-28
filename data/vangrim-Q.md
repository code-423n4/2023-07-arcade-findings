The file `ARCDVestingVault` has several functions where downcasting from `uint256` to `uint128` is done. 

#revokeGrant
`uint256 withdrawable = _getWithdrawableAmount(grant);`
`grant.withdrawn += `uint128(withdrawable)`

#claim
`uint256 withdrawable = _getWithdrawableAmount(grant);`

....

`grant.withdrawn += uint128(withdrawable);`
`grant.withdrawn += uint128(amount);`

Although, in this particular case - the downcasting does not present any potential threat - it is still considered bad practice because of potential overflows.

The recommended mitigation step would be to change the `uint256 withdrawable` to `uint128 withdrawable`. This can be done since the `struct Grant` only has `uin128` variables excluding `latestVotingPower`.


