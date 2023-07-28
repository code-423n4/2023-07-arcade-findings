## Any comments for the judge to contextualize your findings

My findings are based on the following areas:

1. Insecure downcasting
2. Insufficient input validation
3. Broken invariants of the protocol
4. Loss of excessive funds deposited, by the users
5. Malicious Access Control Role being able to `DoS` critical functionality of the protocol
6. Cross chain replay attacks possible since no `chainId` is used for the leafHash calculation in MerkleTree verifications.
7. Discrepancies in the expected behavior of the protocol and actual logic implementations

## Approach taken in evaluating the codebase

I initially started with the `NFTBoostVault` contract. I analyzed how the voting power is delegated to the delegates and how the `ERC1155` NFT deposits could add a multiplier to the existing voting power.
I tried to find any logical errors and arithmetic errors with regard to how the voting power is calculated in the contract.
Then tried to search if any invariant can be broken by either front running key functions of the protocol.
Since ERC1155 and ERC20 tokens are transferred to and from the contract, I tried to find the possibility of reentrancy attacks.
Then i tried to find any access control issues in the critical functions of the protocol.

Then I moved to the `ArcadeTreasury.sol` contract. Here I tried to find attack vectors related to `threshold value` updates and how key invariants of the contract can be broken by changing the order in which functions are called.
This was comparatively a much easier contract to audit since it followed same function template.

Then I audited the `ARCDVestingVault.sol` contract. Here the main focus was on whether the grant allocation can be manipulated. Whether grant revokes can be `DoS`. And how the voting power of a delegate for a particular grant can be manipulated.

Similarly I audited `BaseVotingVault`, `ArcadeTreasury`, `ArcadeTokenDistributor` and `ReputationBadge` contracts in the mentioned order to find if there are any logic related errors and whether I could break any invariants which those contracts are expected to hold.

Remaining contracts were also audited at the later part of the audit. Above mentioned contracts were the main contracts I focused on during the audit.


## Architecture recommendations

The protocol follows an interesting architecture related to the governance. Where the `Voting Vaults` are introduced as depositories to manage voting tokens in the governance system.
The `voting vault` have their independent vote counting mechanisms thus adding a novelty to the protocol implementation. Further the vote delegation is implemented which enables a owner the tokens to delegate their voting power to a respective `delegate`.

Similarly the `Core Voting Contracts` are used to submit and vote on proposed governance proposals. Hence the protocol uses the separate contracts to compute the voting power via `Voting Vaults` and then use them to vote for the proposals in the `Core Voting Contracts`.

The ability to use both `ERC20` and `ERC1155` tokens for governance voting power calculations are also interesting. But one would like to get a better understanding on the eligibility criteria of the `ERC1155` NFTs to be used in the governance.
And what are the requirements to be fulfilled to get a higher multiplier for a particular NFT over another and under what criteria those requirements are chosen.


## Codebase quality analysis

Codebase was comprehensively written. And the order in which certain libraries were used to assist logic implementation within the contracts was commendable.
There should be more attention paid to the `Natspec` comments given in the codebase. Since some of the `Natspec` comments had `typos` in them and some of the comments were not in order.
This could confuse the new auditors and developers who are analyzing the code base.
  

## Centralization risks

There was multiple centralization risks within the contract. For example most of the critical functions of the contracts were controlled by `onlyAdmin` role or `onlyOwner` role.
Hence any `private key` compromise of these privileged users, could negatively affect the functionality of the protocol.

For example :

In the `ArcadeTreasury` contract the key functions such as `ArcadeTreasury.setThreshold` and `ArcadeTreasury.setGSCAllowance` are controlled by the `onlyRole(ADMIN_ROLE)` addresses. Hence any compromise to these addresses could be detrimental to the protocol.
Hence it is recommended a `multisig` is used as the `admin`.

In the `ARCDVestingVault` contract the key functions such as `ARCDVestingVault.addGrantAndDelegate`, `ARCDVestingVault.revokeGrant`, `ARCDVestingVault.deposit` and `ARCDVestingVault.withdraw` are controlled by `onlyManager` modifier. And the `manager` address can be changed by the `timelock` address.
Hence the centralization is with the `manager` address and the `timelock` address since they have the complete authority over how the `grants` are given and revoked. These are critical operations of the protocol.

Other centralization risk was found in the `ArcadeTokenDistributor` contract. Here the critical functions such as `ArcadeTokenDistributor.toTreasury` and `ArcadeTokenDistributor.toCommunityRewards` used to distribute rewards among the `treasury`, `community` and the `team` etc... are controlled by the `onlyOwner` modifier.
Which means the contract deployer has the complete control over when to distribute the reward tokens for the respective parties. Hence if the `owner` is compromised before rewards are distributed then it could be a loss to the participants of the protocol.


## Mechanism review

The procedure in which the `Voting vaults` are used to compute the voting power for the governance is well implemented.
The novel approach taken in the vote-counting mechanism in the `voting vaults` are commendable.
Even though the `Core Voting Contracts` are not largely within the scope of the audit, the demarcation between the vote counting and actual voting on proposals conducted by the `Core Voting Contracts` are commendable.

The idea to enable a multiplier for the `ERC1155` NFTs when computing voting power is an interesting idea. Such mechanisms provide more use cases for the `ERC1155` NFTs and is overall better for the entire ecosystem.
Further it eliminates over dependence on `ERC20` tokens in most of the other governance protocols and provides novel ways to think on how NFTs could be used in other protocols as well. 

## Systemic risks

Systemic risks for the protocol can come from the ability of the privileged users to change the critical parameters of the protocol. Hence it is related to centralization risks as well.
Most of the critical functions are controlled by `onlyOwner` or `onlyAdmin` thus compromise of any of those users could damage the entire system.

Further there needs to be a proper understanding on how the value of `multiplier` is decided based on the quality of the `NFT`. Else it will be confusing for the users on which NFT to deposit in order to increase their share of voting power in the governance.
This could contribute to lack of clarity on the behavior of the protocol thus putting the entire system at a disadvantage

### Time spent:
25 hours