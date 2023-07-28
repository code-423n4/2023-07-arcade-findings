# Approach of the review

My main focus for this security review was primarily on reviewing the `NFTBoostVault` contract and assessing all
possible exploitation scenarios. I began the review by examining the smaller contracts and libraries included in the
scope, before moving on to the larger ones. During this process, I also reviewed all dependencies, such as those in the
`./external`, `./interfaces` and `./libraries` directories, that are not directly included in the scope. Below are my
observations and the knowledge I have gathered about the protocol.

## Observations

`ArcadeToken` is an `ERC20` token that can only be minted from an address with the `MINTER_ROLE`. The token has a
limited initial supply of `100,000,000e18`, which can be increased up to 2% once per year.

The initial supply of the `ArcadeToken` is minted during the deployment _(in the constructor)_ to the
`ArcadeTokenDistributor`, which is responsible for the distribution. This initial supply is distributed by the
`ArcadeTokenDistributor`'s owner among several groups as follows:

-   `ArcadeTreasury` (25.5%)
-   Arcade's development partner (0.6%)
-   Community reward pools (15%)
-   `ArcadeAirdrop` (10%)
-   Arcade's team (16.2%)
-   Arcade's launch partners (32.7%)

`ArcadeAirdrop` receives tokens from the `ArcadeTokenDistributor`. The owner sets a Merkle root with the deposits
encoded as hash `[address, amount]`. Before the expiration of the `ArcadeAirdrop`, users who are included in the Merkle
trie can _`claimAndDelegate`_ a certain amount of tokens _(up to the amount they are included in the Merkle trie)_ from
the contract. The contract then sends and delegates these tokens to the `NFTBootsVault`. Once the `ArcadeAirdrop` is
over, the owner can `reclaim` the remaining tokens.

`NFTBoostVault`

Here are the possible flows to create registration:

1. User calls `addNftAndDelegate` to register and grant voting power to himself, and `transfer` his `erc20` and
   `erc1155` tokens into the `NFTBoostVault`.
2. User calls `addNftAndDelegate` with 0 as `tokenId` and `tokenAddress`, which will apply the default multiplier `1e3`,
   which is possible to be greater than the actual ERC1155 multiplier.
3. `AirdropArcade` directly calls `airdropReceive` and also has multiple possible flows before the erc20 tokens are
   transferred into the vault:
    1. If the user is not registered → create a new registration **but only the default multiplier will be applied**.
    2. If the user is registered → add to his amount and `_syncVotingPower`.

During the registration process, the user can choose another address to which they can delegate their voting power. If
the user does not provide another delegate address, the delegatee will be the user's address.

Additional functionality after the registration is created:

1. Users can change the delegatee address through the `delegate` function.
2. Users can `withdraw` their locked `ArcadeTokens` + `ReputationBadge`.
3. Users can add an additional amount of tokens to their registration to increase the voting power.
4. Users can withdraw only their NFT without the arcade tokens through the `withdrawNft` function.
5. Users can update their NFT. Two possible scenarios:
    1. If the user already has deposited an NFT, the old NFT will be returned, and the new one will take its place.
    2. If the user has not deposited an NFT, the new NFT will be set.
6. Every user can call `updateVotingPower` and pass an array of addresses with a maximum length of 50. The function will
   recalculate these addresses' voting power based on their multiplier and locked tokens.

## Security Interview

**Q:** What in the protocol has value in the market?

**A:** The ArcadeToken and ReputationBadge have value in the market, and they can be used for voting in the governance.

**Q:** In what case can the protocol/users lose money?

**A:** Users can lose money if their tokens have been locked in some of the vaults, or if a malicious user is able to
steal their tokens or NFTs.

**Q:** What are some ways that an attacker can achieve their goals?

**A:** An attacker can achieve their goals by managing to trick the protocol into thinking it is eligible to receive the
airdrop, or by finding a way to enter some of the vaults into an invalid state.


### Time spent:
36 hours