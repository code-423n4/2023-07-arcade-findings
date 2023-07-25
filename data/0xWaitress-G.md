### [G-1]duplicate vault checks on CoreVoting::vote can be done by Openzeppelin Enumerable Set

```solidity
        for (uint256 i = 0; i < votingVaults.length; i++) {
            // ensure there are no voting vault duplicates
            for (uint256 j = i + 1; j < votingVaults.length; j++) {
                require(votingVaults[i] != votingVaults[j], "duplicate vault");
            }
```
this is gas intensive for a large length of votingVaults, consider using [Openzeppelin EnumerableSet](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/structs/EnumerableSet.sol) Set which reduce O(n^2) into O(n)