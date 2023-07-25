### [L-1] _getWithdrawableAmount revert due to division by zero, when block.number is equal to grant.cliff

totalBlocksPostCliff would be zero, which gets used as a divisor for total un
```solidity
    function _getWithdrawableAmount(ARCDVestingVaultStorage.Grant memory grant) internal view returns (uint256) {
        // if before cliff or created date, no tokens have unlocked
        if (block.number < grant.cliff) {
            return 0;
        }
        // if after expiration, return the full allocation minus what has already been withdrawn
        if (block.number >= grant.expiration) {
            return grant.allocation - grant.withdrawn;
        }
        // if after cliff, return vested amount minus what has already been withdrawn
        uint256 postCliffAmount = grant.allocation - grant.cliffAmount;
        uint256 blocksElapsedSinceCliff = block.number - grant.cliff;
        uint256 totalBlocksPostCliff = grant.expiration - grant.cliff;
        uint256 unlocked = grant.cliffAmount + (postCliffAmount * blocksElapsedSinceCliff) / totalBlocksPostCliff;

        return unlocked - grant.withdrawn;
    }
```

### [N-1] calldatas in proposal in CoreVoting can be grouped into struct to avoid length checks 

```solidity
    function proposal(
        address[] calldata votingVaults,
        bytes[] calldata extraVaultData,
        address[] calldata targets,
        bytes[] calldata calldatas,
        uint256 lastCall,
        Ballot ballot
    ) external {
        require(targets.length == calldatas.length, "array length mismatch");
        require(targets.length != 0, "empty proposal");
```

### [N-2] assert statement in GSCVault would use up all remaining gas which is not ideal for user who mis-configured; use require is strictly better 
```solidity

    function proveMembership(
        address[] calldata votingVaults,
        bytes[] calldata extraData
    ) external {
        // Check for call validity
        assert(votingVaults.length > 0);
```

### [N-3] cliff to be bigger than startTime does not make sense when startTime is set to block.number

consider if the manager wants to add a position by startTime == 0, which means start now, and cliff is 0 so as to start vesting immediately, the cliff cant really be set since the startTime is subject to when the block gets mined. Consider adds a condition to set cliff to block.number too, if cliff is 0.

```solidity
        if (startTime == 0) {
            startTime = uint128(block.number);
        }
        // grant schedule check
        if (cliff >= expiration || cliff < startTime) revert AVV_InvalidSchedule();
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L109

### [N-4] cliffAmount checks does not allow a cliffAmount which equals to amount is problematic
this line blocks atomic release, which means the token simply gets immediately available at cliff.

```solidity
if (cliff >= expiration || cliff < startTime) revert AVV_InvalidSchedule();
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L112