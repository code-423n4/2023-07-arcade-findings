# Non-Critical Issues

## [N-01] - Unchecked Initialization of `arcadeToken` in `ArcadeTokenDistributor.sol`

In the `ArcadeTokenDistributor.sol` contract, there is a potential issue with the `arcadeToken` variable being used for transfers without being properly initialized. The relevant lines of code causing this concern are:

```solidity
arcadeToken.safeTransfer(_devPartner, devPartnerAmount);
```

If the `arcadeToken` variable is not set during the contract's construction or through a setter function, calling functions on it, like `safeTransfer`, can lead to an "invalid address" error.

### Mitigation Recommendation

To address this issue, it is recommended to set the `arcadeToken` variable during contract deployment through the constructor and declare the variable as `immutable`. This ensures that the `arcadeToken` is always initialized with a valid address before any operations are performed on it.

## [N-02] - Consider Using the `delete` Keyword to Reset State Variables

In the `NFTBoostVault.sol` and `ARCDVestingVault.sol` contracts, there are instances where state variables are manually set to zero. Instead of manually setting each state variable, consider using the `delete` keyword to reset them to their default values.

Example code snippets:
[NFTBoostVault.sol#L251](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L251-L254)
[ARCDVestingVault.sol#L178](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L178-L185)

```solidity
// NFTBoostVault.sol
function withdraw(uint128 amount) external override nonReentrant {
         ...
          {
            registration.amount = 0;
            registration.latestVotingPower = 0;
            registration.withdrawn = 0;
            registration.delegatee = address(0);
          }
         ...
}

// ARCDVestingVault.sol
function revokeGrant(address who) external virtual onlyManager {
      ...
        // delete the grant
        grant.allocation = 0;
        grant.cliffAmount = 0;
        grant.withdrawn = 0;
        grant.created = 0;
        grant.expiration = 0;
        grant.cliff = 0;
        grant.latestVotingPower = 0;
        grant.delegatee = address(0);
    }
```

### Mitigation Recommendation

Instead of manually setting the state variables to zero, you can use the `delete` keyword to reset them to their default values. This will make the code cleaner and less error-prone.

## [N-03] - Unnecessary Type Change

In `ArcadeTreasury.sol`, there is an unnecessary type conversion when checking the token type:

```solidity
function _spend(address token, uint256 amount, address destination, uint256 limit) internal {
    // ...
    // transfer tokens
    if (address(token) == ETH_CONSTANT) {
    // ...
    }
    // ...
}
```

The variable `token` is already of type `address`, and there is no need to explicitly convert it to `address` again.

### Mitigation Recommendation

To improve code clarity and avoid unnecessary type conversion, you can directly compare the `token` variable with `ETH_CONSTANT` without the type conversion:

```solidity
function _spend(address token, uint256 amount, address destination, uint256 limit) internal {
    // ...
    // transfer tokens
    if (token == ETH_CONSTANT) {
    // ...
    }
    // ...
}
```

By making this change, the code remains more concise and easier to understand.