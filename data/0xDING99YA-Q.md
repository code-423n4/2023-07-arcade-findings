 ## [L-01] Users should still be allowed to withdraw tokens from ArcadeAirdrop.sol after the expiration time.
 Consider enable the users to withdraw tokens after expiration, current docs only specified governance can collect unclaimed tokens after expiration, not mention users can't claim the tokens after expiration.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/libraries/ArcadeMerkleRewards.sol#L79

    if (block.timestamp > expiration) revert AA_ClaimingExpired();
