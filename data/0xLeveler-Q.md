## [L-01] External calls in an un-bounded for-loop may result in a DOS
```solidity
    for (uint256 i = 0; i < targets.length; ++i) {
            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
            (bool success, ) = targets[i].call(calldatas[i]);
    }
```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L333-L345


## [L-02] Missing target-existence Checks Before Low-level Calls
The current code only checks if the length of calldatas array provided match the length of targets, Low-level calls return success if there is no code present at the specified address.
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L333-L345



## [L-03] Lack of Specificity in Batch Call Error-handling
The function `batchCalls` in `ArcadeTreasury.sol` uses the call function to execute external calls on several targets. However, this method returns a boolean value, indicating the success or failure of the call, but it does not provide any specific error messages per call in the batch. This lack of specific error handling might make it challenging to diagnose and fix issues.
If one call in the batch fails everything reverts.

```solidity 
    function batchCalls(
        address[] memory targets,
        bytes[] calldata calldatas
    ) external onlyRole(ADMIN_ROLE) nonReentrant {
        if (targets.length != calldatas.length) revert T_ArrayLengthMismatch();
        // execute a package of low level calls
        for (uint256 i = 0; i < targets.length; ++i) {
            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
            (bool success, ) = targets[i].call(calldatas[i]);
            // revert if a single call fails
            if (!success) revert T_CallFailed();
        }
    }
```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L333-L345


## [L-04] Possible Array Duplicates in Batch calls may have unintended side effects
The target contracts might have internal interactions or dependencies that are sensitive to the order in which the functions are called. Executing duplicate targets with different calldatas could trigger unintended side effects due to the specific order of execution.
If the duplicate targets have different calldatas, they might execute different functions with varying parameters. This could lead to inconsistent changes in the state of the target contracts, resulting in unexpected or undesirable behavior.
Consider using a different data structure to prevent duplicate targets from being included in the batch in the first place.
Arrays should be checked for duplicates.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L333-L345


## [L-05] Partial Execution can cause potential inconsistencies
If a call to one of the targets fails due to incorrect or incompatible calldata, the entire batch execution will not revert since the call function does not revert on failure. As a result, some calls might succeed while others fail, leading to partial execution and potential inconsistencies.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L333-L345
