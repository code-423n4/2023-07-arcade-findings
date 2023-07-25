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