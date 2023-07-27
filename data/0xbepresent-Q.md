1 - The `ARCDVestingVault.delegate()` should validate the `to` parameter is not a zero address
==

The [ARCDVestingVault.delegate()](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L260) function does not validate that the `to` parameter is not zero. The voting power will be lost if the `to` parameter is zero

2 - The `ARCDVestingVault.addGrantAndDelegate()` does not validate the `startTime` parameter is not in the past
==

The [ARCDVestingVault.addGrantAndDelegate()](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L91C14-L91C33) function does not validate the `startTime` parameter. The manager can set the startTime parameter in the past causing that tokens to be unlocked instantly. It could be worth to add a validation so the manager can not setup a `grant.startTime` in the past.


3 - It is possible to transfer the `ReputationBadge` and create a market for them
==

The [ReputationBadge.mint()](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L98) is for users who meet certain criterias. As the [documentation](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L29) says:

```
 * Reputation badges are ERC1155 tokens that can be minted by users who meets certain criteria.
 * For example, a user who has completed a certain number of tasks can be awarded a badge.
```

So users who won those NFTs can sell (transfer to anyone) them in the market and the people who obtain them can get voting power without meeting any criteria. If the protocol wants to maintain the reputation NFT for only the people who meets certain criteria, it should block the NFT transfers.


4 - There is no any function to help to remove the approvals in the `ArcadeTreasury.sol` contract
==

The [ArcadeTreasury.sol_approve()](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L384) helps to approve tokens for a spender.

The problem is that there is no any function to help to remove approvals. It will not be possible to remove spender's approvals if for some reason the `ArcadeTreasury` needs to remove the approval to an specific spender.

The `ArcadeTreasury` should allow the spender's approve zero amount in order to remove a spender approval.
