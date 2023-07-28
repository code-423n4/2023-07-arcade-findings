### 1. In ArcadeTreasury.sol, the cool-down period restrictions can be bypassed, by updating setThreshold with a thresholds.small value below current allowance.

In `setThreshold()`, when the new `thresholds.small` value is less than existing GSC allowance `gscAllowance[token]`, `gscAllowance` will be reset. This bypass the mandatory cool-down period control in `setGSCAllowance()`.

I consider this low severity because Admin role should be considered trusted in setting critical parameters. However, whenever `setThreshold()` is called with a smaller threshold, this side effect of updating gscAllowance before a cool-down period ending might be considered undesirable.

```solidity
//ArcadeTreasury.sol - setThreshold()
...
        // if gscAllowance is greater than new small threshold, set it to the new small threshold
        if (thresholds.small < gscAllowance[token]) {
|>          gscAllowance[token] = thresholds.small;

            emit GSCAllowanceUpdated(token, thresholds.small);
        }
...

```

```solidity
//ArcadeTreasury.sol - setGSCAllowance()
...
        // enforce cool down period
        if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {
            revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);
        }
...

```

Recommendation:
Consider add a cool-down period check inside `setThreshold()` as well.

### 2. No minimal Multiplier value check in setMultiplier() function
In NFTBoostVault.sol, a manager can set a multiplier with a maximum of 1.5e3, and the minimum multiplier value should be 1e3. However, there are only checks for max. value is enforced in `setMultiplier()` and nothing to prevent a value lower than 1e3 to be set.

```solidity
    function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {
        if (multiplierValue > MAX_MULTIPLIER) revert NBV_MultiplierLimit();

        NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];
        // set multiplier value
        multiplierData.multiplier = multiplierValue;

        emit MultiplierSet(tokenAddress, tokenId, multiplierValue);
    }

```

Recommendation:
Add a check for min value, in case a value lower than 1e3 is accidentally passed.

### 3. No checks for minimal spending threshold values enforced in spend or approve functions
In ArcadeTreasury.sol, by design there are three threshold of spending or funds approval for governance or GSC spending - small, medium and large. However, only the upper bound for each level of spending is enforced, no lower bound is enforced.  This means that governance can use `largeSpend()` or `mediumSpend()` to execute small amount of spending.

This creates overlaps between multiple functions including `smallSpend()`, `mediumSpend()`, `largeSpend()`, `approveSmallSpend()`,`approveMediumSpend()`,`approveLargeSpend()`. Under some conditions, multiple functions can be interchangeable for users causing confusion.

```solidity
    function mediumSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
        if (destination == address(0)) revert T_ZeroAddress("destination");
        if (amount == 0) revert T_ZeroAmount();

        _spend(token, amount, destination, spendThresholds[token].medium);
    }
```
```solidity
    function _spend(address token, uint256 amount, address destination, uint256 limit) internal {
        // check that after processing this we will not have spent more than the block limit
        uint256 spentThisBlock = blockExpenditure[block.number];
        if (amount + spentThisBlock > limit) revert T_BlockSpendLimit();
        blockExpenditure[block.number] = amount + spentThisBlock;
...
```
Recommendation:
Consider adding minimal spending threshold checks in medium and large spending functions, to ensure clear distinctions in spending limit tiers.