## Summary

|       | Issue        | Instances    |
| :---: |:-------------|:------------:|
| 1 | State variables should be packed to save storage slots | 2 |
| 2 | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 2 |
| 3 | `storage` variable should be cached into `memory` variables instead of re-reading them | 5 |
| 4 | Use `unchecked` blocks to save gas | 2 |


## Findings

### 1- State variables should be packed to save storage slots :

State variables inside contract should be packed if possible to save storage slots and thus will save gas cost.

There are 2 instances of this issue:

* Instance 1 :

```
File: token/ArcadeToken.sol

94      address public minter;
97      uint256 public mintingAllowedAfter;
```

As the variable `mintingAllowedAfter` represent a timestamp value, it can't realistically overflow 2^96, so it should be packed with the address variable `minter` (which only takes 20 bytes), this will **save 1 storage slots**, the code should be modified as follow :

```
/// @notice Minter contract address responsible for minting future tokens
address public minter;

/// @notice The timestamp after which minting may occur
uint96 public mintingAllowedAfter;
```

* Instance 2 :

```
File: token/ArcadeTokenDistributor.sol

34      bool public treasurySent;
39      bool public devPartnerSent;
44      bool public communityRewardsSent;
49      bool public communityAirdropSent;
54      bool public vestingTeamSent;
59      bool public vestingPartnerSent;
```

All the variables in the code above are of type boolean which only takes 1 bit in storage, so they can all be packed together to save gas **save 5 storage slots**, the code should be rearranged as follows :

```
/// @notice The Arcade Token contract to be used in token distribution.
IArcadeToken public arcadeToken;
    
 /// @notice A flag to indicate if the treasury has already been transferred to
bool public treasurySent;
/// @notice A flag to indicate if the token development partner has already been transferred to.
bool public devPartnerSent;
/// @notice A flag to indicate if the token development partner has already been transferred to.
bool public devPartnerSent;
/// @notice A flag to indicate if the community airdrop contract has already been transferred to
bool public communityAirdropSent;
/// @notice A flag to indicate if the launch partners have already been transferred to
bool public vestingTeamSent;
/// @notice A flag to indicate if the Arcade team has already been transferred to
bool public vestingPartnerSent;

/// @notice 25.5% of initial distribution is for the treasury
uint256 public constant treasuryAmount = 25_500_000 ether;

/// @notice 0.6% of initial distribution is for the token development partner
uint256 public constant devPartnerAmount = 600_000 ether;

/// @notice 15% of initial distribution is for the community rewards pool
uint256 public constant communityRewardsAmount = 15_000_000 ether;

/// @notice 10% of initial distribution is for the community airdrop contract
uint256 public constant communityAirdropAmount = 10_000_000 ether;

/// @notice 16.2% of initial distribution is for the Arcade team
uint256 public constant vestingTeamAmount = 16_200_000 ether;

/// @notice 32.7% of initial distribution is for Arcade's launch partners
uint256 public constant vestingPartnerAmount = 32_700_000 ether;
```


### 2- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 2 instances of this issue :

**File: ReputationBadge.sol** [Line 51-58](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L51-L58)
```solidity
52:     mapping(uint256 => bytes32) public claimRoots;
55:     mapping(uint256 => uint48) public claimExpirations;
58:     mapping(uint256 => uint256) public mintPrices;
```


**File: ArcadeTreasury.sol** [Line 58-65](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L58-L65)
```solidity
59:     mapping(address => uint48) public lastAllowanceSet;
62:     mapping(address => SpendThreshold) public spendThresholds;
65:     mapping(address => uint256) public gscAllowance;
```

### 3- `storage` variable should be cached into `memory` variables instead of re-reading them :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There are 5 instance of this issue :

**File: ARCDVestingVault.sol** [Line 145-148](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L145-L148)

In the code linked above the value of `grant.delegatee` is read multiple times (3) from storage and it's value does not change so it should be cached into a memory variable in order to save gas (using `delegatee` value) by avoiding multiple reads from storage.

**File: ARCDVestingVault.sol** [Line 145-148](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L145-L148)

In the code linked above the value of `grant.delegatee` is read multiple times (3) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: ARCDVestingVault.sol** [Line 262-269](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L262-L269)

In the code linked above the value of `grant.delegatee` is read multiple times (4) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: ARCDVestingVault.sol** [Line 268-281](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L268-L281)

In the code linked above the value of `grant.latestVotingPower` is read multiple times (4) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: ARCDVestingVault.sol** [Line 344-356](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L344-L356)

In the code linked above the value of `grant.delegatee` is read multiple times (3) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.


### 4- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 2 instances of this issue:

**File: ARCDVestingVault.sol** [Line 141](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L141)
```solidity
unassigned.data -= amount;
```

In the code linked above the operation cannot underflow because of the check in line [115](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L115).

**File: ARCDVestingVault.sol** [Line 215](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#215)
```solidity
unassigned.data -= amount;
```

In the code linked above the operation cannot underflow because of the check in line [213](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#213).