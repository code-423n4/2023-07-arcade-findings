## 1. Summary of Codebase
## 1.1 Description
[Arcade.xyz](https://www.arcade.xyz/) is a borrow lending platform where NFTs are escrowed as collateral and its valuation is put up for loans. This audit focuses on the Arcade.xyz token-based on-chain governance system built with reference to the [Council Framework](https://docs.element.fi/governance-council/council-protocol-overview). 

The audit in scope can be separated into the following mechanisms:
- Voting Vaults
- Reputation Badge (NFT for boosting voting power)
- Token Contracts

###  1.2 Voting Vaults
Voting vaults are vaults where governance tokens are deposited to gain voting power for voting and/or proposing proposals. It supports delegation of full voting power to other parties, but does not support partial delegations.

### 1.2.1 NFTBoostVault
This is the general, core community voting vault for governance, where a NFT (`ReputationBadge.sol`) minted to voter can be attached to voting power to boost votes depending on multiplier set by protocol admins. Here is a quick summary 

Entry
- `addNftAndDelegate()`: Deposit arcade governance tokens to vault with an option to add a owned ReputationBadge NFT that boosts votes by multiplier set by protocol admins. If not, the base multiplier is 1 and amount of governance tokens deposited will equate to voting power assigned to user himself or his chosen delegatee (automatically defaults to user if no delegatee is passed)

Adjustments
- `addTokens()`: Allows additional deposit of governanace tokens to increase voting power of user himself or delegatee

<br/>

- `airDropReceive()`: Allows airdrop contract to add more tokens to a user to increase voting power of user himself or delegatee

<br/>

- `delegate()`: Allows regisration owner that originally deposited governance tokens to change delegatee that receives the user's voting power

<br/>

- `updateNFT()`: Allows regisration owner that originally deposited governance tokens to change the attached NFT, presumably to increase multiplier. Votes are updated after multiplier assigned to NFT is registered.

Exit
- `withdraw()`: Allows full or partial withdrawals of governance tokens, with voting power subsequently decreased. If user performs a full withdrawal, all voting power is lost and his registration (voting details) is deleted.

### 1.2.2 ARCDVestingVault
This is a specialized voting vault designed for early Arcade team members, contributors, and dev partners, that holds tokens in escrow subject to a vesting timeline. Both locked and unlocked tokens held by the vault contribute governance voting power, Since locked tokens are held by the ARCDVestingVault, they are not eligible for NFT boosts.

Entry
- `addGrantAndDelegate()`: Manager only function that sets a grant where voting power will be delegated to (either grant reciepient himself or his chosen delegatee). There is a cliff amount of tokens to encourage grant recipients to stay committed to the Arcade governance where a portion or `cliff` amount of tokens is periodically unlocked for grant recipient usage

Adjustments
- `revokeGrant()`: At any point of time, manager can revoke the full grant of grant recipient or any remaining unclaimed grant tokens, causing grant recipient or delegatee to lose all remaining voting power

<br/>

- `delegate()`: Allows grant recipient to change delegatee that receives his voting power assigned by manager

<br/>

Exit
- `claim()`: Allows grant recipient to fully or partially claim unlocked `cliff` tokens after cliff period has passed. Else it allows partial or full claiming of all tokens after grant expiration (vesting) period ends. The voting power of grant recipient or his delegatee is then decremented

### 1.2.3 ImmutableVestingVault
This vault is exactly identical to ARCDVestingVault, with the exception that grants cannot be revoked by `revokeGrant()` once it is assigned via `addGrantAndDelegate()`. This is presumably for founders/important stakeholders of Arcade protocol.

### 1.2.4 ArcadeGSCVault
This is a specialized voting vault for voters that have met the minimum voting power set by the Arcade governance to be part of the Arcade governance General Steering Council (GSC). 

Entry
- `proveMembership()`: For voters that have met the minimum threshold set by arcade governance, they will be able to be a member of Arcade governance GSC. This is achieved through:
    - Receiving a grant by manager via Vesting Vaults
    - Deposition of Arcade governance token into community NFTBoostVault
    - Delegation by other users via any type of vaults

Exit
- `kick()`: Any GSC member can be kicked by anybody once total voting power queried via registered vaults falls below minimum thresholds. This can be because of:
    - A grant being revoked by manager
    - Votes being re-delegated away from GSC memeber by owners of arcade governance tokens
    - Withdrawal/Claiming of arcade governance tokens by GSC member


## 1.2 Reputation Badge
This is simply an NFT that any whitelisted voter can pay to mint and then added to the general community NFTBoostVault to increase voting power. The whitelisted address and mint price is managed by the `BADGE_MANAGER_ROLE`.

## 1.3. Token Contracts

### 1.3.1 ArcadeToken
This is simply the governance token that is used to determine voting power. Generally, amount of tokens deposited into voting vaults equals the voting power received (excluding multiplier from attached ReputationBadge NFT), with the exception of GSC members who votes in a separate voting contract and receives 1 vote after proving membership status.

### 1.3.2 ArcadeTokenDistributor
This contract simply mints the tokens assigned to each stakeholder, with a initial maximum token supply of 100_000_000e18 tokens.

### 1.3.3 ArcadeAirDrop
This contract allows airdrop of tokens to voters which can increase their voting power in `NFTBoostVault.airDropReceive()`. 10_000_000e18 tokens is assigned for this feature.


### 1.3.4 ArcadeTreasury
This is the main contract where successful proposals from community or GSC voting can spend tokens within the treasury. Any type of ERC20 token and native ETH is supported to be held by treasury as long as spending threshold is set.

It allows small, medium and large approving and spending of tokens, with corresponding small, medium and large spending limits assigned to each token. It also has a block expenditure limit to limit spendings each block for successful proposals. Note: GSC are only authorize to spend or approve smaller spendings.


## 2. Architecture Improvements 

## 2.1 In scope contracts
### 2.1.1 Allow updates of grant amount
Since grant assignment is done by a trusted manager, consider allowing adjustment of grant to avoid having to revoke and assign again if grant amount is assigned wrongly or simply to decrease or increase amount of grants assigned to a grant recipient.

## 2.2 Out of Scope Contracts

### 2.2.1 Consider the addition of the following statuses to CoreVoting contracts
Consider adding a function to describe the status of a proposal and refactor logic of checks based on this statuses. This will make checking proposal status easy and introduce useful features for proposal editing and cancellation, along with fail-safe options to remove malicious proposals.

- Updatable: Consider the addition of a updatable period where proposals details can be updated by proposal creator to avoid having to re-propose proposals in the event changing details is desired. In this period, voting should not be allowed.

<br/>

- Active: Consider the addition of a active period where voting is allowed as long as proposal has not reach its final state, that is cancelled, expired, vetoed or executed.

<br/>

- Expired: Consider the addition of a expired status that indicates the expiry of proposal after expiration timestamp has passed. At this status, proposals can no longer be executed.

<br/>

- Cancelled: Consider adding a cancellation mechanism as a fail safe for proposer's that have change their mind instead of leaving the proposal for expiration and potentially even open up the possibility of execution.

<br/>

- Vetoed: Currently, Arcade has no mechanism to cancel proposal except if proposal has successfully executed or its expiration cause it to be unexecutable. This esentially means that voters would have to actively vote against malicious proposals to prevent malicious proposals from potentially executing. Consider adding a priviledged VETOER role to VETO malicious proposals to esentially cancel them from possible excecution. 

<br/>

- Queued: Consider adding a queuing period before execution as a fail-safe and last check of proposal instead of allowing immediate execution once quorum and for votes are met. In this status, proposal can be cancelled.

<br/>

- Executed: Consider adding a executed status to indicate that proposal have been successfully executed


### 2.2.2 Implement status mechanism to allow checking of status of proposal
Many DAOS have a function to view the current status of the proposal, but this is missing in Arcade's voting contracts. Consider implementing and integrating with UI a feature and function to allow voters to check status of an proposal.


### 2.2.3 Implement a proposal cap for proposers
To avoid spamming of proposals, consider implementing a cap for the number of active proposals a proposer can propose at any given time. Many DAOs only allow a single active proposal at any given time.



## 3. Centralization Risks

## 3.1 Priviledged Roles
Note: Only roles that can intentionally impact voters/users from losing voting power or funds are included

### 3.1.1 MANAGER_ROLE in ARCDVestingVault
- `setMultiplier()`: Can change multiplier of any voter owning a ReputationBadge NFT. Since there is no minimum multiplier check, the multiplier can be set below 100% (less than 1e3)

<br/>

- `addGrantAndDelegate()`: Can add anybody to be assigned a grant giving the grant recipient or delegatee voting power 

<br/>

- `revokeGrant()`: Revoke any grant at any time, preventing claiming of grant tokens and making grant recipient or delegatee lose voting power

<br/>

- `withdraw()`: withdraw governance tokens at anytime, potentially preventing grant recipients from receiving grants assigned

### 3.1.2 Timelock role in voting vaults
- `setManager()`: Can change address of manager role, preventing current manager role from executing privileged functions 

<br/>

- `unlock()`: Can lock or unlock withdrawals in NFTBoostVaults, preventing withdrawals of voter's governance tokens


## 4. Time Spent
- Day 1: Understand the 4 different types of voting vaults
- Day 2: Understand the general community voting contract and NFTBoostVault voting vault mechanisms
- Day 3: Understand GSC mechanisms, voting vault and voting contract
- day 4: Finish up Analysis

### Time spent:
48 hours