```
N01: `votingPower.push()` in ARCDVestingVault.sol can revert. 
Per History.sol: @param data The data to append, should be at most 192 bits and will revert if not.
consider using the SafeCast library and convert to uint192.

```
```
N02: Consider delete grant instead of setting to default values for `revokeGrant()` in ARCDVestingVault.sol.
```

```
N04: Consider using named imports to be more specific and increased readability.
```

```
N05:  `setMultiplier()` should not be able to set multiplierValue below 1e3. Having a reputation badge is meant to increase voting power. Anything below will be slashing voting power.
```

```
N06: Use all cap names for constant variables in ArcadeTokenDistributor.sol
```

```
N07: Use named input param for `revokeGrant()` in ImmutableVestingVault.sol. Also input param missing natspec
```

```
N08: It is recommended to use 2-step verification for `setManager()` in BaseVotingVault.sol 
```

```
N09: Having a single EOA as the only owner(Admin) of contracts is a large centralization risk and a single point of 
               failure. A single private key may be taken in a hack, or the sole holder of the key may become unable to 
               retrieve the key when necessary. Consider changing to a multi-signature setup, or having a role-based 
               authorization model.
```

```
L01 :  NFTBoostVault only supports EOA. ERC1155 tokens sent by a contract without  implementing {IERC1155Receiver-onERC1155Received} will be locked away in the NFTBoostVault. There will be no way to retrieve.
```

```
L02: Although  not likely, uint128(withdrawable) in ARCDVestingVault.sol can possibly overflow. Consider using Safecast Library.
```

```
L03:  add supportsInterface() from ERC-165. Per EIP-1155, Smart contracts implementing the ERC-1155 standard MUST implement the ERC-165 supportsInterface function and MUST return the constant value true if 0xd9b67a26 is passed through the interfaceID argument.
```

