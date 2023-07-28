# Arcade Non-Critical Findings


## [N-01] Zero as a function argument should have a descriptive meaning
Consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.

### 3 Instances
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L152-#L153
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L171
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L286



## [N-02] Use abi.encode rather than abi.encodePacked() in hash function
Using abi.encodePacked can result in hash collision when used in hashing functions
Consider using abi.encode as this pads data to 32 byte segments

### 1 Instances
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L211


## [N-03] Custom error has no error details
In Solidity, the use of custom error messages provides a valuable method of conveying meaningful information about failures during execution. In the current implementation, the custom errors lack specifics, making it challenging to understand the root cause of a failure. It's advisable to incorporate parameters into your error messages to indicate which user action or specific value caused the exception. This not only enhances error transparency but also aids debugging and fosters a more robust and maintainable codebase. Providing such precise error context greatly helps developers identify and resolve issues faster.

### Instances
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L114
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L136
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L174
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L195
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L217
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L236
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L255
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L273
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L305
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L337
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L361
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L387


