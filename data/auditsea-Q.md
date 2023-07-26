Zero remaining amount check is missing in `ARCDVestingVault::revokeGrant()`

Description

If `revokeGrant()` function is called after expiration, `remaining` will be zero and there is no need to call transfer.

```solidity
    function revokeGrant(address who) external virtual onlyManager {
        ... ...

        // transfer the remaining tokens to the vesting manager
        uint256 remaining = grant.allocation - grant.withdrawn; // --> Remaining value can be zero after expiration
        grant.withdrawn += uint128(remaining);
        token.safeTransfer(msg.sender, remaining);

        ...
    }
```

To fix this, we might wrap transfer call like below:
```solidity
    if (remaining > 0) {
        grant.withdrawn += uint128(remaining);
        token.safeTransfer(msg.sender, remaining);
    }
```
