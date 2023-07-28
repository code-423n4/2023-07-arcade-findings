# [L-01] `setThreshold` bypass set allowance cooldown

## Impact
When new threshold is set, GSC allowance is being update according to new small threshold. But cooldown is not applied like `setGSCAllowance`.

## Proof of Concept
```
    function testSetThresholdCooldownNotApplied() public {
        vm.startPrank(governance);
        IArcadeTreasury.SpendThreshold memory thresholds =
            IArcadeTreasury.SpendThreshold({small: 1 ether, medium: 10 ether, large: 100 ether});
        // Threshold is being set
        arcadeTreasury.setThreshold(ETH_CONSTANT, thresholds);
        // Allowance can be set again immediately in the same block after being set by `setThreshold`
        arcadeTreasury.setGSCAllowance(ETH_CONSTANT, 0.9 ether);
        vm.stopPrank();
    }
```

## Recommended Mitigation Steps
Properly add cooldown if allowance is updated during set threshold