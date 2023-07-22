# nft/ReputationBadge.sol:

## cache `_claimData.length` in `publishRoots` function

As its used twice in the function entry, and once every loop iteration, the function should look like the following

```solidity
    function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
        uint256 len = _claimData.length;
        if (len == 0) revert RB_NoClaimData();
        if (len > 50) revert RB_ArrayTooLarge();

        for (uint256 i = 0; i < len; i++) {
            ...

```

## use pre-increment alongside with unchecked {} with the loop counter

`i` cannot overflow because of the condition thanks to the condition `i < _claimData.length`, consequently it is advisable to use `unchecked {}` to omit overflow checks for the variable `i`, with the modification, the function should look like this

```
        for (uint256 i = 0; i < _claimData.length;) {
            ...
            unchecked {
                ++i;
            }

        }

```

## cache `_claimData[i]` and `_claimData[i].tokenId`

as these elements are accessed multiple times throughout the function, after the optimization the loop should look like this

```jsx
	ClaimData Data;
	uint256 _token_id;
for (uint256 i = 0; i < _claimData.length; i++) {
	Data = _claimData[i];
	_token_Id = Data.tokenId;

  // expiration check
  if (Data.claimExpiration <= block.timestamp) {
	  revert RB_InvalidExpiration(Data.claimRoot, _token_Id);
  }

  claimRoots[_token_Id = Data.claimRoot;
  claimExpirations[_token_Id] = Data.claimExpiration;
  mintPrices[_token_Id] = Data.mintPrice;
} 
```

## Final version

all in all the last version of the function should look somehow like this

```jsx
function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
        uint256 len = _claimData.length;
        if (len == 0) revert RB_NoClaimData();
        if (len > 50) revert RB_ArrayTooLarge();

        for (uint256 i = 0; i < _claimData.length;) {
					Data = _claimData[i];
					_token_Id = Data.tokenId;
				
				  // expiration check
				  if (Data.claimExpiration <= block.timestamp) {
					  revert RB_InvalidExpiration(Data.claimRoot, _token_Id);
				  }
				
				  claimRoots[_token_Id = Data.claimRoot;
				  claimExpirations[_token_Id] = Data.claimExpiration;
				  mintPrices[_token_Id] = Data.mintPrice;

						unchecked {
                ++i;
            }
        }

        emit RootsPublished(_claimData);
    }
```

# [token/ArcadeTokenDistributor.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol)

## grouping state variables to optimize storage slots

re-arranging the following `bool` variables to be declared right after each others will result in the contract using 5 less stage slots in the EVM:

- [treasurySent](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L34C17-L34C29)
- [devPartnerSent](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L39)
- [communityRewardsSent](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L44)
- [communityAirdropSent](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L49)
- [vestingTeamSent](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L54)
- [vestingPartnerSent](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L59C17-L59C35)

additionally it will make subsequent reads to any variable in the same slot consume less gas (cold reads vs hot reads)

all what has to be done is changing the order of variables from :

```jsx
		/// @notice 25.5% of initial distribution is for the treasury
    uint256 public constant treasuryAmount = 25_500_000 ether;
    /// @notice A flag to indicate if the treasury has already been transferred to
    bool public treasurySent;

    /// @notice 0.6% of initial distribution is for the token development partner
    uint256 public constant devPartnerAmount = 600_000 ether;
    /// @notice A flag to indicate if the token development partner has already been transferred to.
    bool public devPartnerSent;

    /// @notice 15% of initial distribution is for the community rewards pool
    uint256 public constant communityRewardsAmount = 15_000_000 ether;
    /// @notice A flag to indicate if the community rewards pool has already been transferred to
    bool public communityRewardsSent;

    /// @notice 10% of initial distribution is for the community airdrop contract
    uint256 public constant communityAirdropAmount = 10_000_000 ether;
    /// @notice A flag to indicate if the community airdrop contract has already been transferred to
    bool public communityAirdropSent;

    /// @notice 16.2% of initial distribution is for the Arcade team
    uint256 public constant vestingTeamAmount = 16_200_000 ether;
    /// @notice A flag to indicate if the launch partners have already been transferred to
    bool public vestingTeamSent;

    /// @notice 32.7% of initial distribution is for Arcade's launch partners
    uint256 public constant vestingPartnerAmount = 32_700_000 ether;
    /// @notice A flag to indicate if the Arcade team has already been transferred to
    bool public vestingPartnerSent;
```

 

to

```jsx
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

    /// @notice A flag to indicate if the treasury has already been transferred to
    bool public treasurySent;
    /// @notice A flag to indicate if the token development partner has already been transferred to.
    bool public devPartnerSent;
    /// @notice A flag to indicate if the community rewards pool has already been transferred to
    bool public communityRewardsSent;
    /// @notice A flag to indicate if the community airdrop contract has already been transferred to
    bool public communityAirdropSent;
    /// @notice A flag to indicate if the launch partners have already been transferred to
    bool public vestingTeamSent;
    /// @notice A flag to indicate if the Arcade team has already been transferred to
    bool public vestingPartnerSent;
```

# [token/ArcadeToken.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol)

## grouping state variables to optimize storage slots

re-arranging the following variables to be declared right after each other will result in the contract using 1 less storage slot in the EVM:

- [MIN_TIME_BETWEEN_MINTS](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L80)
- [minter](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L94)

additionally it will make subsequent reads to any variable in the same slot consume less gas (cold reads vs hot reads)

all what has to be done is changing the order of variables from :

```jsx
		uint48 public constant MIN_TIME_BETWEEN_MINTS = 365 days;

		/* 
			* declaration of other variables *
		*/

    /// @notice Minter contract address responsible for minting future tokens
    address public minter;
```

to:

```jsx
		uint48 public constant MIN_TIME_BETWEEN_MINTS = 365 days;
    /// @notice Minter contract address responsible for minting future tokens
    address public minter;

		/* 
		* declaration of other variables *
		*/
```

# [ArcadeTreasury.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol)

## using unchecked, and caching the array size and elements

we can cache the length of `calldatas` and  `targets[i]` in [batchCalls](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L333) such as it becomes like the following

```jsx
function batchCalls(
	address[] memory targets,
	bytes[] calldata calldatas
) external onlyRole(ADMIN_ROLE) nonReentrant {
	uint256 call_len = calldatas.length;
	uint256 target_len = targets.len;
	if (target_len != call_len) revert T_ArrayLengthMismatch();
	address target;
	// execute a package of low level calls
	for (uint256 i = 0; i < target_len; ++i) {
		target = targets[i];
		if (spendThresholds[target].small != 0) revert T_InvalidTarget(target);
		(bool success, ) = target.call(calldatas[i]);
	  // revert if a single call fails
	  if (!success) revert T_CallFailed();
	}
}
```

# [NFTBoostVault.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol)

## using unchecked, and caching the array size and elements

we can cache the length of `userAddresses` and  `userAddresses[i]` in [updateVotingPower](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L342) such as it becomes like the following

```jsx
function updateVotingPower(address[] calldata userAddresses) public override {
	uint256 len = userAddresses.length;
	if (len > 50) revert NBV_ArrayTooManyElements();
	address _userAddress;

	for (uint256 i = 0; i < len; ) {
		_userAddress = userAddresses[i];
		NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[_userAddress];
		_syncVotingPower(_userAddress, registration);
		unchecked {
			++i;
		}
	}
}
```