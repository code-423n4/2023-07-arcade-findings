## Limits should be inclusive 
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146 - `block.timestamp` is compared to a param called `mintingAllowedAfter` which means that if `block.timestamp` is equal to `mintingAllowedAfter` the function should revert

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L308 - same here but with cooldown period

## Missing events on important parameters change
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L68-L72
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L80-L84
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L174-L178

## Extra ether is not returned to the user
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L109C1-L109C83
if a user calls mint() with `msg.value` grater than `mintPrice` his eth will be locked in the contract until(and if) the admin(`BADGE_MANAGER_ROLE`) decides to refund them.

## Unsafe downcast
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L166
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L171
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L242
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L244
The values are unsafely downcasted and truncated from uint256 to uint128. It is safer to use openzeppelin SafeCast.