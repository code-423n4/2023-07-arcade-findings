# Airdrop claimer cannot use address(0) to assign voting power to self when claim like in `addNftAndDelegate`

## Impact
ArcadeMerkleRewards `claimAndDelegate` revert if `delegate` is address(0) so claimer can't pass address(0) to `airdropReceive` to default delegate to self like in `addNftAndDelegate`. If claimer want to delegate to self they must pass their own address.

## Proof of Concept
```
    function testAirdropClaimerCannotDelegateToSelfWithZeroAddress() public {
        vm.startPrank(airdropReceiver);
        // Can't do
        vm.expectRevert(abi.encodeWithSignature("AA_ZeroAddress(string)", "delegate"));
        arcadeAirdrop.claimAndDelegate(address(0), 1 ether, new bytes32[](0));
        // This is fine
        arcadeAirdrop.claimAndDelegate(airdropReceiver, 1 ether, new bytes32[](0));
    }
```

## Recommended Mitigation Steps
Remove address(0) check during `claimAndDelegate`

# User can lock arbitrary ERC1155 and make their voting power 0

## Impact
During registration via `addNftAndDelegate`, vault will reject the registration if the supplied ERC1155 has 0 multiplier. But user can still lock an ERC1155 that has 0 multiplier via `updateNft` and make their voting power 0. Technically user can lock arbitrary contract that implements ERC1155 interface.

## Proof of Concept
```
    function testUpdateRandomERC1155MakeVotingPowerZero() public {
        arcadeToken.approve(address(nftBoostVault), 1 ether);
        nftBoostVault.addNftAndDelegate(1 ether, 0, address(0), address(0));
        assertEq(nftBoostVault.queryVotePower(address(this), block.number, ""), 1 ether);

        uint256 tokenId = 1;
        mockERC1155.mint(address(this), tokenId, 1, "");
        mockERC1155.setApprovalForAll(address(nftBoostVault), true);
        nftBoostVault.updateNft(uint128(tokenId), address(mockERC1155));
        assertEq(nftBoostVault.queryVotePower(address(this), block.number, ""), 0);
    }
```

## Recommended Mitigation Steps
Validate multiplier > 0 when `updateNft`
```
    function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {
        if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);

        if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();

        if (getMultiplier(newTokenAddress, newTokenId) == 0) revert NBV_NoMultiplierSet();
```

# ReputationBadge can have tokenId 0 or larger than uint128 which NFTBoostVault doesn’t support

## Impact
Badge issuer can accidentally issue NFT with tokenId 0 or larger than uint128 that doesn't work with NFTBoostingVault

## Recommended Mitigation Steps
Validate tokenId when publish root
```
    function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
        ...
        for (uint256 i = 0; i < _claimData.length; i++) {
            // expiration check
            if (_claimData[i].claimExpiration <= block.timestamp) {
                revert RB_InvalidExpiration(_claimData[i].claimRoot, _claimData[i].tokenId);
            }
            // tokenId check
            if (_claimData[i].tokenId == 0 || _claimData[i].tokenId > type(uint128).max) {
                revert RB_InvalidTokenId();
            }
```

# Delegate should check for registration/grant existent

## Impact
Currently delegate of both NFTBoostVault and ARCDVestingVault doesn't check for registration/grant existent. Allowing user who doesn't have rights to delegation to call the function. Creating inconsistent state where delegatee could be set while everything else is still empty. For NFTBoostVault if user accidentally delegate before register they won't be able to register via normal method. Instead they would have to add tokens and update nft separately.

## Recommended Mitigation Steps
Check for registration/grant existent before allowing delegation and revert if not exist.

# User can addNftAndDelegate with either arbitrary tokenAddress or tokenId

## Impact
Since `addNftAndDelegate` only validate multipliers and transfer if both tokenAddress and tokenId are valid (both not 0), if one of them is non-zero it will pass the validation and registration state would use those value.

## Proof of Concept
```
    function testRegisterArbitraryTokenAddress() public {
        arcadeToken.approve(address(nftBoostVault), 1 ether);
        nftBoostVault.addNftAndDelegate(1 ether, 0, address(1234), address(0));
        NFTBoostVaultStorage.Registration memory registration = nftBoostVault.getRegistration(address(this));
        assertEq(registration.tokenAddress, address(1234));
    }
```

## Recommended Mitigation Steps
Validate that both must be zero or non-zero at once