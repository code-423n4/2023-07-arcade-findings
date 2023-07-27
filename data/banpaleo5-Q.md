# Table of Contents
| Number | Issues Details                                                                                         | Count |
| :----- | :----------------------------------------------------------------------------------------------------- | :---- |
| [L-1] | Implement Reentrancy Guard for the `withdraw` Function | 2 |
| [L-2] | Handle Unused `receive()` Function to Prevent Ether Lock | 1 |
| [L-3] | Adopt `safeTransfer` for Safer Token Transfers | 2 |
| [N-1] | Eliminating Redundant Casts | 1 |
| [N-2] | Emitting Events for Significant Parameter Changes | 7 |
| [N-3] | Adding Previous Value to Events | 1 |
| [N-4] | Placing the `nonReentrant` Modifier First | 2 |
| [N-5] | Where appropriate, multiple address mappings can be consolidated into a single mapping of an address to a struct | 1 |


## [L-1]</a><a name="L-1"> Implement Reentrancy Guard for the `withdraw` Function

To prevent reentrancy attacks and ensure the safety of funds in your contract, it is essential to implement a reentrancy guard in the `withdraw` function. Without a proper guard, malicious contracts or attackers can exploit potential vulnerabilities and repeatedly call the `withdraw` function to drain funds.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
function _withdrawNft() internal {
        
        NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

        if (registration.tokenAddress == address(0) || registration.tokenId == 0)
            revert NBV_InvalidNft(registration.tokenAddress, registration.tokenId);

        
        IERC1155(registration.tokenAddress).safeTransferFrom(
            address(this),
            msg.sender,
            registration.tokenId,
            1,
            bytes("")
        );

        
        registration.tokenAddress = address(0);
        registration.tokenId = 0;

        
        _syncVotingPower(msg.sender, registration);
    }
```

```solidity
File: ReputationBadge.sol
function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {
        if (recipient == address(0)) revert RB_ZeroAddress("recipient");

        
        uint256 balance = address(this).balance;

        
        
        payable(recipient).transfer(balance);

        emit FeesWithdrawn(recipient, balance);
    }
```
</details>




## [L-2]</a><a name="L-2"> Handle Unused `receive()` Function to Prevent Ether Lock

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ArcadeTreasury.sol
receive() external payable {}
```
</details>



## [L-3]</a><a name="L-3"> Adopt `safeTransfer` for Safer Token Transfers

Consider using the `safeTransfer` function instead of the `transfer` function for safer token transfers within your contract.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: ARCDVestingVault.sol
token.transferFrom(msg.sender, address(this), amount);
```

```solidity
File: NFTBoostVault.sol
token.transferFrom(from, address(this), amount);
```
</details>


## [N-1]</a><a name="N-1"> Eliminating Redundant Casts

It appears that there is a redundant cast in the code. The variable is already of the same type as the one it is being cast to, making the cast unnecessary. Removing this redundant cast enhances code readability and eliminates unnecessary operations.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ArcadeTreasury.sol
address(token)
```
</details>



## [N-2]</a><a name="N-2"> Emitting Events for Significant Parameter Changes

It is important to emit events when modifying state variables to enable effective monitoring using off-chain monitoring tools. Emitting events facilitates the tracking of activities related to critical parameter changes.

<details>
<summary><i>There are 7 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
function setAirdropContract(address newAirdropContract) external override onlyManager {
```

```solidity
File: BadgeDescriptor.sol
function setBaseURI(string memory newBaseURI) external onlyOwner {
```

```solidity
File: BaseVotingVault.sol
function setManager(address manager_) external onlyTimelock {
```

```solidity
File: ReputationBadge.sol
function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {
```

```solidity
File: BaseVotingVault.sol
function setTimelock(address timelock_) external onlyTimelock {
```

```solidity
File: ArcadeTreasury.sol
function setGSCAllowance(address token, uint256 newAllowance) external onlyRole(ADMIN_ROLE) {
```

```solidity
File: NFTBoostVault.sol
function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {
```
</details>



## [N-3]</a><a name="N-3"> Adding Previous Value to Events

In the codebase, events are commonly emitted to track significant changes made to the contracts. However, certain events are missing crucial parameters. It is recommended to enhance these events by including the previous original value in addition to the new value. This addition provides valuable context and facilitates the monitoring and analysis of contract modifications.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ArcadeToken.sol
event MinterUpdated(address newMinter);
```
</details>



## [N-4]</a><a name="N-4"> Placing the `nonReentrant` Modifier First

To adhere to best practices and ensure consistent code organization, it is recommended to position the `nonReentrant` modifier before all other modifiers. This placement emphasizes the importance of preventing reentrancy and promotes a standardized ordering of modifiers for improved readability.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
function airdropReceive(
        address user,
        uint128 amount,
        address delegatee
    ) external override onlyAirdrop nonReentrant {
```

```solidity
File: ArcadeTreasury.sol
function mediumSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
```
</details>



## [N-27]</a><a name="N-27"> Prefer Explicit Symbol Imports over Relative Path Imports

Using relative path imports in the current form is discouraged as it can introduce unpredictable namespace pollution. The Solidity documentation recommends adopting explicit symbol imports as a best practice. By specifying imported symbols explicitly, code clarity and maintainability can be improved. For more information, refer to the [Solidity documentation on importing other source files](https://docs.soliditylang.org/en/v0.8.15/layout-of-source-files.html#importing-other-source-files).

<details>
<summary><i>There are 8 instances of this issue:</i></summary>

```solidity
File: ArcadeTreasury.sol
import "./interfaces/IArcadeTreasury.sol";
```

```solidity
File: ImmutableVestingVault.sol
import "./ARCDVestingVault.sol";
```

```solidity
File: ArcadeGSCCoreVoting.sol
import "./external/council/CoreVoting.sol";
```

```solidity
File: ARCDVestingVault.sol
import "./external/council/libraries/History.sol";
import "./external/council/libraries/Storage.sol";
import "./libraries/ARCDVestingVaultStorage.sol";
import "./libraries/HashedStorageReentrancyBlock.sol";
import "./interfaces/IARCDVestingVault.sol";
import "./BaseVotingVault.sol";
```

```solidity
File: BaseVotingVault.sol
import "./external/council/libraries/History.sol";
import "./external/council/libraries/Storage.sol";
import "./libraries/HashedStorageReentrancyBlock.sol";
import "./interfaces/IBaseVotingVault.sol";
```

```solidity
File: ArcadeToken.sol
import "../interfaces/IArcadeToken.sol";
```

```solidity
File: ArcadeGSCVault.sol
import "./external/council/vaults/GSCVault.sol";
```

```solidity
File: NFTBoostVault.sol
import "./external/council/libraries/History.sol";
import "./external/council/libraries/Storage.sol";
import "./libraries/NFTBoostVaultStorage.sol";
import "./interfaces/INFTBoostVault.sol";
import "./BaseVotingVault.sol";
```
</details>



## [G-1]</a><a name="G-1"> Use assembly language for `address(0)` checks for better gas efficiency

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ArcadeTokenDistributor.sol
function toCommunityRewards(address _communityRewards) external onlyOwner {
```
</details>



## [G-2]</a><a name="G-2"> When it's impossible for them to overflow, such as in for- and while-loops, `++i` and `i++` should be written as `unchecked{++i}` and `unchecked{i++}` respectively for better efficiency

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
for (uint256 i = 0; i < userAddresses.length; ++i) {
```

```solidity
File: ArcadeTreasury.sol
for (uint256 i = 0; i < targets.length; ++i) {
```
</details>



## [G-3]</a><a name="G-3"> Employing the `delete` statement can contribute to gas conservation

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ARCDVestingVault.sol
grant.allocation = 0;
grant.cliffAmount = 0;
grant.withdrawn = 0;
grant.created = 0;
grant.expiration = 0;
grant.cliff = 0;
grant.latestVotingPower = 0;
```
</details>



## [G-4]</a><a name="G-4"> Employing `uints` or `ints` that are smaller than 32 bytes (256 bits) leads to overhead costs

<details>
<summary><i>There are 4 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
uint128 public constant MAX_MULTIPLIER = 1.5e3;
```

```solidity
File: NFTBoostVault.sol
uint128 public constant MULTIPLIER_DENOMINATOR = 1e3;
```

```solidity
File: NFTBoostVault.sol
uint128 multiplier = 1e3;
```

```solidity
File: NFTBoostVault.sol
uint128 newVotingPower = (_amount * multiplier) / MULTIPLIER_DENOMINATOR;
```

```solidity
File: NFTBoostVault.sol
uint128 locked = registration.amount - registration.withdrawn;
```

```solidity
File: ArcadeTreasury.sol
uint48 public constant SET_ALLOWANCE_COOL_DOWN = 7 days;
```

```solidity
File: ReputationBadge.sol
uint48 claimExpiration = claimExpirations[tokenId];
```

```solidity
File: ArcadeToken.sol
uint48 public constant MIN_TIME_BETWEEN_MINTS = 365 days;
```
</details>



## [G-5]</a><a name="G-5"> The operation `<x> += <y>` consumes more gas than `<x> = <x> + <y>`

<details>
<summary><i>There are 5 instances of this issue:</i></summary>

```solidity
File: ARCDVestingVault.sol
grant.withdrawn += uint128(withdrawable);
grant.withdrawn += uint128(amount);
```

```solidity
File: ReputationBadge.sol
amountClaimed[recipient][tokenId] += amount;
```

```solidity
File: NFTBoostVault.sol
balance.data -= amount;
registration.withdrawn += amount;
```

```solidity
File: NFTBoostVault.sol
balance.data += _amount;
```

```solidity
File: ARCDVestingVault.sol
unassigned.data -= amount;
```
</details>



## [G-6]</a><a name="G-6"> Declaring constants as `private` instead of `public` leads to gas savings

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
uint128 public constant MAX_MULTIPLIER = 1.5e3;
uint128 public constant MULTIPLIER_DENOMINATOR = 1e3;
```

```solidity
File: ArcadeTokenDistributor.sol
uint256 public constant treasuryAmount = 25_500_000 ether;
uint256 public constant devPartnerAmount = 600_000 ether;
uint256 public constant communityRewardsAmount = 15_000_000 ether;
uint256 public constant communityAirdropAmount = 10_000_000 ether;
uint256 public constant vestingTeamAmount = 16_200_000 ether;
uint256 public constant vestingPartnerAmount = 32_700_000 ether;
```
</details>



## [G-7]</a><a name="G-7"> Instead of using `true` and `false` boolean states, it's more efficient to use `uint256(1)` and `uint256(2)` respectively

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ArcadeTokenDistributor.sol
treasurySent = true;
```

```solidity
File: ArcadeTokenDistributor.sol
devPartnerSent = true;
```

```solidity
File: ArcadeTokenDistributor.sol
communityRewardsSent = true;
```

```solidity
File: ArcadeTokenDistributor.sol
communityAirdropSent = true;
```

```solidity
File: ArcadeTokenDistributor.sol
vestingTeamSent = true;
```

```solidity
File: ArcadeTokenDistributor.sol
vestingPartnerSent = true;
```
</details>



## [N-5]</a><a name="N-5"> Where appropriate, multiple address mappings can be consolidated into a single mapping of an address to a struct

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: ArcadeTreasury.sol
mapping(address => uint48) public lastAllowanceSet;
mapping(address => SpendThreshold) public spendThresholds;
mapping(address => uint256) public gscAllowance;
```
</details>