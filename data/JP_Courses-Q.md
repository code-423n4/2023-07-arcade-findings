0. NFTBoostVault.sol: Seems possible to add an invalid delegatee address? e.g. any arbitrary contract address including any of the ArcadeDAO governance contracts. Maybe no risk but probably shouldn't be allowed:

https://github.com/code-423n4/2023-07-arcade/blob/34b2821a40d4e73f70056cbd675bba079e66f05e/contracts/NFTBoostVault.sol#L122


1. The function allows an attacker/rogue user to use multiple user(msg.sender) addresses to register himself multiple times as a voter/delegatee because he uses a different registration struct instance and storage location each time, which could lead to gaining more voting power than intended/permitted and potentially influencing governance decisions unfairly/intentionally, as part of an attack vector, even if there are limits in place per voter to limit their voting power/influence. Centralisation of voting power risk?

https://github.com/code-423n4/2023-07-arcade/blob/34b2821a40d4e73f70056cbd675bba079e66f05e/contracts/NFTBoostVault.sol#L451-L510


2. Seems to be no reason to use public visibility modifier, should use external instead:

https://github.com/code-423n4/2023-07-arcade/blob/34b2821a40d4e73f70056cbd675bba079e66f05e/contracts/NFTBoostVault.sol#L362

function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {

Recommendation:

function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) external override onlyManager {
