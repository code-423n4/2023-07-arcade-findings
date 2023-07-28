# Analysis

## Introduction

This technical analysis report delves into the underlying smart contracts that constitute the token-based governance system of Arcade.xyz. This report explores various contract categories, including Voting Vaults, Core Voting Contracts, Token, and NFT, while highlighting insights and potential attack vectors in the governance ecosystem.

## Architecture

1. Voting Vaults Contracts Inheritance Graph:

![Vesting Vaults](https://res.cloudinary.com/davyfibzy/image/upload/v1690553946/nogvvqu3t5mdkvip2zxk.png)

2. Core Voting Contracts Inheritance Graph:

![Core Voting](https://res.cloudinary.com/davyfibzy/image/upload/v1690553800/axavsr5jmwccr4om3qjz.png)

## NFTs

Arcade.xyz uses ERC1155 tokens to grant multipliers to users' voting power. `ReputationBadge.sol` contract consists the majority of the Core logic accompanying by tokenURI descriptor contract.

**Contracts:**

1. `ReputationBadge.sol`: This is the main contract which helps users mint the NFTs by providing merkle proofs. This NFT Badge can be used in governance to give a multiplier to a user's voting power.
2. `BadgeDescriptor.sol`: Basic descriptor contract for badge NFTs which helps setting and getting `tokenURI`.

### Insights

* Badge Manager is supposed to add claim data first which will be validated when user mints the NFTs.
* There is no Check for `_claimData[i].tokenId` to be not equal to zero.
* In case, a user sent more than the `mintPrice`, then the remain amount is not refunded to the User.
* Badge Manager can Withdraw all the Eth fees anytime which includes any excess pay done by the user.
* Users have option to claim the Claimable amount in 1 transaction or can claim over time. But it should be claimed before the expiration time.

### Possible Attack Vectors

* `_mint` is used instead of `_safeMint` which can lead to loss of NFTs incase user's address doesn't accept ERC1155.
* In Case, Badge Manager adds `_claimData[i].tokenId` = 0, then the User who claims that NFTs will not be able to get `multiplier` in their voting rights.
* `.transfer` used which uses hardcoded gas. `.call` should be used instead with a check for reentrancy.

## Tokens

This contracts are responsible for the token's initial deployment and distribution, including the airdrop and initial distributor contracts. It provides users with voting power and influence.

**Contracts:**

1. `ArcadeToken.sol`: This contract is an ERC20 token implementation for the Arcade token [ARCD] with an inflationary cap of 2% per year.
2. `ArcadeTokenDistributor.sol`: This contract is used for distribution of Arcade Tokens to the various Stake holders of the Protocol.
3. `ArcadeAirdrop.sol`: This contract receives tokens from the ArcadeTokenDistributor and facilitates airdrop claims.

### Insights

* Initially, 100 Million `ARCD` Tokens are minted to `ArcadeTokenDistributor.sol`.
* There is a Cooldown period of 365 days between mints such that no tokens can be minted in between.
* Mint function has an inflation capped at 2% per year.
* Every function inside `ArcadeTokenDistributor` can be called just once that too only by the owner.
* For Airdrops, in case no one claims, Owner can reclaim it but only after expiration.

### Possible Attack Vectors

* Use of `_mint` instead of `_safeMint` in `ArcadeToken` contract.
* In `ArcadeTokenDistributor`, if by mistake owner call any other function before `setToken` function, then `ARCD` tokens which were supposed to be send through that function will be frozen forever.

## Arcade Treasury

This contract hold the treasury funds and manages the amount that needs to be approved or spend with 3 threshold levels which includes: large amount, medium amount and small amount.

### Insights

* Thresholds are modifiable via the governance timelock (ADMIN role).
* `CORE_VOTING_ROLE` can call spend functions with custom quorums based on the spend thresholds.
* The GSC can spend within its allowance for each token, which is updated by the ADMIN role.
* In case new threshold.small is less than the current allowance for GSC, then the current allowance will be slashed to new threshold.small value.
* Allowance updates have a 7-day cooldown and cannot exceed the small threshold to require governance votes for larger spends.

### Possible Attack Vectors

* No protection against Potential Huge return data values returning through `call` made in `batchCalls`.
* Wrong Implementation of `blockExpenditure` for a particular Block number. As per Natspec, it was supposed to limit the max tokens that can be spent/approved in a single block for a particular threshold. But Implementation differs from that.
* In case of Tokens with Multiple addresses, Cooldown period can be bypassed.

## ARCDVestingVault

This contract is a vesting vault which help creating and managing vesting grants for Arcade tokens over time. `ImmutableVestingVault` is also a `ARCDVestingVault` but without the functionality to revoke a grant.

### Graph Representation

![Graph](https://res.cloudinary.com/davyfibzy/image/upload/v1690555651/uikr1zygtzlaa586bo9p.png)

### Insights

* The manager can add or remove grants and is changeable via the timelock.
* The manager can deposit the tokens to increase the `unassigned` amount and withdraw only the left over `unassigned` amount after giving grant.
* Grants have a delegatee address that receives voting power, updatable by the grant recipient.
* Three time parameters define each grant: "created," "cliff," and "expiration" block numbers.
* Tokens are unlocked gradually from the "cliff" block to the "expiration" block.
* No emergency withdrawal exists, and only deposited funds are recoverable.
* In case of revoke, Funds upto that particular time is sent to the grant owner and remaining funds are transferred back to manager instead of adding it to `unassigned` number.
* User have option to use `queryVotePower` and clear everything more than `staleBlockLag` into the past.

### Possible Attack Vectors

First of all, Code is written really well. Still hypothetical attack vector possible here:

* In case, value of `staleBlockLag` is very low, then a grant owner can potentially do a Griefing Attack where he calls `queryVotePower` to clear everything more than `staleBlockLag` into the past which might lead to voting power becoming `0`. And because of this, there will be DoS in `_syncVotingPower` resulting in DoS in `claim` and `revokeGrant` function.

## NFTBoostVault

This contract enables Users to enhance their voting power in governance through ERC1155 NFT multipliers.

### Insights

* Users deposit ERC20 tokens and provide ERC1155 NFTs as calldata to confirm ownership.
* Their Voting power multiplier will depend on the `multiplierData.multiplier` value set by the Manager for that particular NFT, ID pair.
* Users can register or update their registration with more token through Airdrop Contract.
* Option to Withdraw or add new tokens or NFTs.
* Voting Power will be sync corresponding to the action performed.
* No emergency withdrawal exists, and funds not sent via `addNftAndDelegate()` are unrecoverable.
* Once entire registration amount is withdrawn, NFT will be withdrawn too and user will need to register again before doing any other operation.

### Possible Attack Vectors

* `ERC1155` tokens outside the ones set in Multiplier Set are not allowed. But in UpdateNFT function, it is possible to lock any random ERC1155 in the contract because of lack of check.
* Same attack vector as `ARCDVestingVault` exist in case `staleBlockLag` is very low.
* Trusting `Timelock` contract to call `unlock`. In case, it doesn't user won't be able to withdraw tokens.

### Time spent:
16 hours