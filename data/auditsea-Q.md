- Zero remaining amount check is missing in `ARCDVestingVault::revokeGrant()`

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

- Can transfer 0 eth and result in wasting gas.
As balance zero checking is missing in the code below, sending 0 eth might happen when mint price is optional.
[ReputationBadge.sol#L163-L174](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L163-L174)