## (low) `NFTBoostVault.withdraw()` shouldn 't revert when `withdrawable < amount`

`NFTBoostVault.withdraw()` shouldn 't revert when `withdrawable < amount` , because user maybe not sure the `amount` value, I suggest set `amount = withdrawable ` when `withdrawable < amount`

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L236

## (low) `ArcadeTreasury.setThreshold()` setting may cause `thresholds.large == thresholds.medium == thresholds.small`

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L276
As we can see, the function just check `<` ,not `=`, and the large ,medium and small should be designed diffrent and `large>medium>small`
```solidity
        if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small) {//@audit （low）要大于等于吧？
            revert T_ThresholdsNotAscending();
        }
```
So I suggest fix it to `thresholds.large <= thresholds.medium || thresholds.medium <= thresholds.small`


## (low) `ReputationBadge.mint()` does not check if `amount == 0`

`ReputationBadge.mint()` does not check if `amount == 0`, it may cause calling repeatly and waste gas.
I suggest check if `amount == 0` and revert.
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L105