## [L-1] User reputation badge claim can be lost when there are multiple consecutive claims
In the `ReputationBadege.sol` an nft is allocated to be claimed using the merkle root and the mapping `amountClaimed[recipient][tokenid]` keeps track of that a user cannot claim greater than a certain amount of nfts of specific tokenId.

But the problem that could rise is that in the following code :
```solidity
 function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
        uint256 mintPrice = mintPrices[tokenId] * amount;
        uint48 claimExpiration = claimExpirations[tokenId];

        if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));

        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
        if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
            revert RB_InvalidClaimAmount(amount, totalClaimable);
        }

        // increment amount claimed
        amountClaimed[recipient][tokenId] += amount;

        // mint to recipient
        _mint(recipient, tokenId, amount, "");
    }
```

Lets say for the first time there is tokenId 10 and allocated nfts are 2. Alice comes and mint 1 first and than 2nd later. All good for this time. But lets say for the same token id alice is allocated another 2. So this time the claimable are again 2. But the `amountClaimed` for token id is already 2. So the alice will not be able to claim the later nfts.
And the revert will happen on the following lines:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L111-L113

Submitting this as low, as there are few assumptions, this can be mitigated if a certain user is not given the same id over different allocations.

Another thing could be that the `totalClaimable` withe the same tokenid in merkle tree could be inclusive of last allocation. That second time the total claimable is 4, 2 for the last time that have been claimed and 2 for this time.

Another way could be when setting the root reset the amount claimed mapping.

Upto judge and sponser to decide whether these considerations were in mind or not, severity could be upgraded.

## [L-2] No check if the large threshold and medium threshold are equal or both are equal to small one
In ArcadeTreasury.sol we have following checks when threshold are being set:
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L276C1-L278C10
```solidity
        if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small) {
            revert T_ThresholdsNotAscending();
        }
```
To ensure that these are in ascending order from small to large. But these checks are under constrained as they do not check if the medium and large are equal or both are equal to small threshold which is unintended. This code needs to be refactored to meet requirement for all the constraints.

## [L-3] Cool down check should be inclusive of the deadline

In `ArcadeTreasury.sol` when setting the gsc allowance the cooldown period check should be inclusive of the deadline so instead of using < use the <= for proper time matching.

```solidity
    function setGSCAllowance(address token, uint256 newAllowance) external onlyRole(ADMIN_ROLE) {
        if (token == address(0)) revert T_ZeroAddress("token");
        if (newAllowance == 0) revert T_ZeroAmount();

        // @audit - cool down deadline check should be inclusive use <=
        if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {
            revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);
        }

        uint256 spendLimit = spendThresholds[token].small;
        // new limit cannot be more than the small threshold
        if (newAllowance > spendLimit) {
            revert T_InvalidAllowance(newAllowance, spendLimit);
        }

        // update allowance state
        lastAllowanceSet[token] = uint48(block.timestamp);
        gscAllowance[token] = newAllowance;

        emit GSCAllowanceUpdated(token, newAllowance);
    }
```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L308C1-L310C10

## [L-4] Changing the minter role should ideally be a two step process
In arcade token `ArcadeToken.sol` a minter can change the minter and assign it to someone else, as it is a very crucial role and is at the code of the token and eco system functionality. This role should be changed in two step instead of one in the following function:
```solidity
     */
    function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

        minter = _newMinter;
        emit MinterUpdated(minter);
    }
```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132C2-L138C1
Implement a two step process like `ownable2Step.sol` by openzeppelin.

## [L-5] Extra eth send with the mint function for the badge nft are not refunded.
In the mint function of `ReputationBadge.sol` extra eth sent are not refunded back to the user and can be lost.
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L98C3-L120C6
```solidity
    function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
        uint256 mintPrice = mintPrices[tokenId] * amount;
        uint48 claimExpiration = claimExpirations[tokenId];

        if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));
        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
        if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
            revert RB_InvalidClaimAmount(amount, totalClaimable);
        }

        // increment amount claimed
        amountClaimed[recipient][tokenId] += amount;

        // mint to recipient
        _mint(recipient, tokenId, amount, "");
    }
```

Implement the refund mechanism instead of extra eth are sent than required.


## [L-6] There should be certain amount of check for the expiration 
In the `ArcadeAirdrop.sol` expiration is passed into constructor and than into `ArcadeMerkleRewards.sol` constructor which is being inherited. But there the checks on the expiration are under constrained, as it is set in the immediate next second of the block.timestamp and expire in next second. There should be minimum time period of expiration defined as constant and expiration should be at least that much or more in the future.
```solidity
// ArcadeAirdrop.sol
    constructor(
        address _governance,
        bytes32 _merkleRoot,
        IERC20 _token,
        uint256 _expiration,
        INFTBoostVault _votingVault
    ) ArcadeMerkleRewards(_merkleRoot, _token, _expiration, _votingVault) {
        if (_governance == address(0)) revert AA_ZeroAddress("governance");

        owner = _governance;
    }
```

```solidity
// ArcadeMerkleRewards.sol
    constructor(bytes32 _rewardsRoot, IERC20 _token, uint256 _expiration, INFTBoostVault _votingVault) {
        if (_expiration <= block.timestamp) revert AA_ClaimingExpired();
        if (address(_token) == address(0)) revert AA_ZeroAddress("token");
        if (address(_votingVault) == address(0)) revert AA_ZeroAddress("votingVault");

        rewardsRoot = _rewardsRoot;
        token = _token;
        expiration = _expiration;
        votingVault = _votingVault;
    }
```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeAirdrop.sol#L42C1-L53C1



## [NC-1] Use the given library functions for setting the values instead of manually setting them
In the `ARCDVestingVault.sol` for updating the assigned value, it is done manually using the pointer :

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L141
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L200

Instead a better approach could be to use the defined setter functions in the `Storage.sol`
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/libraries/Storage.sol#L85-L87

```solidity
   function set(Uint256 storage input, uint256 to) internal {
        input.data = to;
    }
```
This could improve consistency and readability of the code.

## [NC-2] Implement the setter functions in the self implemented storage contracts.
There are some storage contract from the council that are used, that have setter functions for setting the values, but the storage contract written by arcade team miss those functions result in lack of consistency across the codebase:

[NFTBoostVaultStorage.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/libraries/NFTBoostVaultStorage.sol)
[NFTBoostVaultStorage.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/libraries/NFTBoostVaultStorage.sol)

## [NC-3] Transfers should only be perform when the values are greater than 0
In the `evokeGrant()` function in `ARCDVestingValut.sol` transfers are being performed even when `withdrawable` and `remainning` values are 0, which is unncessary. Refactor the following code to only perform transfers for non zero values

```solidity
function revokeGrant(address who) external virtual onlyManager {
        // load the grant
        // grants have been set above.
        ARCDVestingVaultStorage.Grant storage grant = _grants()[who];

        // if the grant has already been removed or no grant available, revert
        if (grant.allocation == 0) revert AVV_NoGrantSet();

        // get the amount of withdrawable tokens
        // tokens will be withdrawable when for a certain who, certain time have passed and tokens have been vested.
        uint256 withdrawable = _getWithdrawableAmount(grant);
        // @note - only do it when withdrawable is not zerro
        grant.withdrawn += uint128(withdrawable);
        token.safeTransfer(who, withdrawable);

        // transfer the remaining tokens to the vesting manager
        uint256 remaining = grant.allocation - grant.withdrawn;
        // @note - QA, only do it when remainning is not zero
        grant.withdrawn += uint128(remaining);
        token.safeTransfer(msg.sender, remaining);

        // update the delegatee's voting power
        _syncVotingPower(who, grant);

        // delete the grant
        grant.allocation = 0;
        grant.cliffAmount = 0;
        grant.withdrawn = 0;
        grant.created = 0;
        grant.expiration = 0;
        grant.cliff = 0;
        grant.latestVotingPower = 0;
        grant.delegatee = address(0);
    }
```

## [NC-4] Use the same reentrancy prevention method across the codebase.
There are two kind of reentrancy libraries being used across the project.

One is standard openzeppelin `ReentrancyGuard.sol` in `ArcadeTreasury.sol`

and other one is `HashedStorageReentrancyBlock.sol` written using the storage pointer way. And it is used in multiple file e.g `BaseVotingVault.sol` and `ARCDVestingVault.sol`

Better use one and openzeppelin is perferable as it is battle tested.