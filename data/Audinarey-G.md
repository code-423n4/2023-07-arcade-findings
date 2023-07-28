- In the ```ARCDVestingVault.sol``` contract, enhance the logic in the ```_syncVotingPower(...)``` function to ensure ```grant.latestVotingPower = newVotingPower;``` state update is done only when ```_syncVotingPower(...)``` is called when revoking a grant.

- In the ```ReputationBadge.sol``` contract, when calling ```publishRoots(...)``` ensure to cache ```_claimData.length``` and ```_claimData[i].tokenId``` as shown below.
```
    function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
        uint256 _claimDataLength = _claimData.length
        if (_claimDataLength == 0) revert RB_NoClaimData();
        if (_claimDataLength > 50) revert RB_ArrayTooLarge();

        for (uint256 i = 0; i < _claimData.length; i++) {
            
            ...

            uint256 ithClaimDataTokenId = _claimData[i].tokenId
            claimRoots[ithClaimDataTokenId] = _claimData[i].claimRoot;
            claimExpirations[ithClaimDataTokenId] = _claimData[i].claimExpiration;
            mintPrices[ithClaimDataTokenId] = _claimData[i].mintPrice;
        }

        emit RootsPublished(_claimData);
    }
```