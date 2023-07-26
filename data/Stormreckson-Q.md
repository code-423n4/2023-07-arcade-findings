https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L304

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L269-L270

1. The `setGSCAllowance` function checks if a cooldown period has passed before updating the allowance for the Small threshold.
However, the `setThreshol`d function does not seem to include this cooldown check. This means the Admin could potentially bypass the cooldown period by invoking `setThreshold` directly, allowing them to update the allowance without adhering to the cooldown restrictions. To address this, it would be advisable to include the cooldown check in the `setThreshold` function as well.

2. https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L333-L334

The `batchCalls` function lacks input validations for the `target` and `calldata` array parameters, Without proper input validation, it's possible for these parameters to contain unexpected or invalid values, which could lead to undesired behavior. It would be advisable to implement input validation to ensure the correctness and integrity of the input values.

3.    https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L113-L116

   constructor (address _minter, address 
   _initialDistribution) ERC20("Arcade", "ARCD") 
   ERC20Permit("Arcade") {
        if (_minter == address(0)) revert 
   AT_ZeroAddress("minter");
        if (_initialDistribution == address(0)) revert 
   AT_ZeroAddress("initialDistribution");
 
In the constructor,checking if `_minter` and `_initialDistribution` addresses are not zero addresses. However, you should use the `require` statement instead of `if` conditions with revert statements. The `require` statement is a more conventional and readable way to handle such validations.