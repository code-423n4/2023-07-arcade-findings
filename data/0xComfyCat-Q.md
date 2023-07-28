# [L-01] Airdrop claimer cannot use address(0) to assign voting power to self when claim like in `addNftAndDelegate`

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