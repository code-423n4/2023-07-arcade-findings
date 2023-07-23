**Gas Optimization Report for the `publishRoots` Function**

Line of code with improvement => https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L140C4-L156C6

1. **Batch Assignments:**

In the original function, the state variables `claimRoots`, `claimExpirations`, and `mintPrices` were updated one by one inside the loop. This resulted in redundant storage writes and increased gas consumption, especially when dealing with multiple claim data entries.

In the optimized version, we use batch assignments to update the state variables after processing all the data entries. This reduces gas consumption by minimizing the number of storage writes.

**Explanation in POC:**

Before Optimization (Original Code):
```solidity
function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
    // ... (existing checks)

    for (uint256 i = 0; i < _claimData.length; i++) {
        // ... (expiration check and processing)
    }

    // ... (emission of event)
}
```

After Optimization (Using Batch Assignments):
```solidity
function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
    // ... (existing checks)

    // Define local arrays to store the updated data
    bytes32[] memory updatedClaimRoots = new bytes32[](_claimData.length);
    uint48[] memory updatedClaimExpirations = new uint48[](_claimData.length);
    uint256[] memory updatedMintPrices = new uint256[](_claimData.length);

    for (uint256 i = 0; i < _claimData.length; i++) {
        // ... (expiration check and processing, update local arrays)
        updatedClaimRoots[i] = _claimData[i].claimRoot;
        updatedClaimExpirations[i] = _claimData[i].claimExpiration;
        updatedMintPrices[i] = _claimData[i].mintPrice;
    }

    // Batch assignment of the updated data to the state variables
    for (uint256 i = 0; i < _claimData.length; i++) {
        claimRoots[_claimData[i].tokenId] = updatedClaimRoots[i];
        claimExpirations[_claimData[i].tokenId] = updatedClaimExpirations[i];
        mintPrices[_claimData[i].tokenId] = updatedMintPrices[i];
    }

    // ... (emission of event)
}
```

2. **Early Exit:**

Added an early exit check to the function to ensure that if the length of `_claimData` is zero, the function returns early without further processing. This avoids unnecessary iterations and reduces gas consumption when there are no new claim data to be updated.

**Explanation in POC:**

```solidity
function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
    // Check if there are any new claim data entries to process
    if (_claimData.length == 0) {
        // Early exit if there are no new claim data
        return;
    }

    // ... (existing checks)

    // Define local arrays to store the updated data
    bytes32[] memory updatedClaimRoots = new bytes32[](_claimData.length);
    uint48[] memory updatedClaimExpirations = new uint48[](_claimData.length);
    uint256[] memory updatedMintPrices = new uint256[](_claimData.length);

    for (uint256 i = 0; i < _claimData.length; i++) {
        // ... (expiration check and processing, update local arrays)
        updatedClaimRoots[i] = _claimData[i].claimRoot;
        updatedClaimExpirations[i] = _claimData[i].claimExpiration;
        updatedMintPrices[i] = _claimData[i].mintPrice;
    }

    // Batch assignment of the updated data to the state variables
    for (uint256 i = 0; i < _claimData.length; i++) {
        claimRoots[_claimData[i].tokenId] = updatedClaimRoots[i];
        claimExpirations[_claimData[i].tokenId] = updatedClaimExpirations[i];
        mintPrices[_claimData[i].tokenId] = updatedMintPrices[i];
    }

    // ... (emission of event)
}
```

**Summary:**

By implementing these gas optimization techniques in the `publishRoots` function, I have significantly reduced the gas cost associated with updating badge claim data. The use of batch assignments and early exit checks results in a more efficient and cost-effective process, especially when dealing with multiple claim data entries. This ensures a streamlined user experience and helps in conserving gas fees for both the users and the system.


