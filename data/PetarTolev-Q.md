# QA Report for contest Arcade.xyz

## Overview

During the audit, 2 low and 4 non-critical issues were found.

## Low Risk Findings Overview

Total: 3 instances over 2 issues

| ID                                                                                            | Finding                                                                              | Instances |
| --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | --------- |
| [L-01](#users-have-no-incentive-to-set-an-nft-token-with-a-multiplier-lower-from-the-default) | Users have no incentive to set an NFT token with a multiplier lower from the default | 2         |
| [L-02](#implementing-require-statement-to-restrict-token-id-0-for-managers)                   | Implementing Require Statement to Restrict TokenId 0 for Managers                    | 1         |

## Non-critical Findings Overview

Total: 23 instances over 4 issues

| ID                                                                         | Finding                                                            | Instances |
| -------------------------------------------------------------------------- | ------------------------------------------------------------------ | --------- |
| [N-01](#unused-storage-variable)                                           | Unused storage variable                                            | 1         |
| [N-02](#introduing-storage-variable-constant-names-instead-of-using-magic) | Introducing storage variable constant names instead of using magic | 19        |
| [N-03](#unused-imports)                                                    | Unused imports                                                     | 1         |
| [N-04](#not-following-the-checks-effects-interactions-pattern)             | Not following the Checks-Effects-Interactions pattern              | 1         |

---

# Low Risk Findings

### Users have no incentive to set an NFT token with a multiplier lower from the default

##### Description

The maximum allowed multiplier that can be set for the `ReputationBadge` from the manager in the protocol is `1.5e3`.
However, there is no incentive for users to `updateNft()` with a `ReputationBadge` token that has a multiplier lower
than `1e3`, because the default multiplier is `1e3`. Doing so would decrease their voting power.

##### Recommendation

Consider if it is possible to remove the default `1e3` multiplier from the `_registerAndDelegate` and `getMultiplier`.

<details>

<summary>Instances (2)</summary>

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L423

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L469

</details>

### Implementing Require Statement to Restrict TokenId 0 for Managers

##### Description

Managers can set `tokenId` to 0. After that, when calling `getMultiplier` for token with `tokenId` 0, it will return
`1e3`, which is the default multiplier value.

##### Recommendation

It is recommended to add a require statement to restrict managers from setting `tokenId` to 0.

<details>
<summary>Instances (1)</summary>

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L363-L371

</details>

# Non-critical Findings

### Unused storage variable

##### Description

In the `NFTBoostVault`'s constructor, an initialized storage variable is set but not used.

##### Recommendation

Remove it if it is unnecessary.

<details>
<summary>Instances (1)</summary>

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L91

</details>

### Introducing storage variable constant names instead of using magic

##### Description

To improve readability and prevent mistakes, consider introducing storage variable constant names instead of using magic
strings while working with the `Storage` pointers library.

##### Recommendation

```diff
+string public constant lockedStorageName = "locked";

constructor(
  IERC20 token,
  uint256 staleBlockLag,
  address timelock,
  address manager
) BaseVotingVault(token, staleBlockLag) {
  ...
-  Storage.set(Storage.uint256Ptr("locked"), 1);
+  Storage.set(Storage.uint256Ptr(lockedStorageName), 1);
}

function getIsLocked() public view override returns (uint256) {
-  return Storage.uint256Ptr("locked").data;
+  return Storage.uint256Ptr(lockedStorageName).data;
}

```

<details>
<summary>Instances (19)</summary>

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L91-L95

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L380

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L393

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L406

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L446

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L541

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L688

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L70-L72

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L371

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L382

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L71

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L83

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L149

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L160

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L171

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L181

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/libraries/HashedStorageReentrancyBlock.sol#L31

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/external/council/vaults/LockingVault.sol#L45

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/external/council/vaults/LockingVault.sol#L65

</details>

### Unused imports

##### Description

Unused imports.

##### Recommendation

Remove all unused imports.

<details>
<summary>Instances (1)</summary>

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/libraries/HashedStorageReentrancyBlock.sol#L5

</details>

### Functions state mutability can be restricted to pure

##### Description

Functions state mutability can be restricted to pure

##### Recommendation

```diff
-function _timelock() internal view returns (Storage.Address storage) {
+function _timelock() internal pure returns (Storage.Address storage) {
  return Storage.addressPtr("timelock");
}
```

<details>
<summary>Instances (2)</summary>

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L159

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L170

</details>

### Not following the Checks-Effects-Interactions pattern

##### Description

Regarding the `NFTBoostVault.updateNft` function, even though there is a `nonReentrant` modifier, the CEI pattern is not
followed. The `_syncVotingPower` logic is executed after the `_lockNft`, which contains an external call with a
callback.

##### Recommendation

It is recommended to always change the state first before making external calls. While the code is not currently
vulnerable due to the `nonReentrant` modifier, it is still best practice to follow the CEI pattern.

```diff
function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {
  ...
-  _lockNft(msg.sender, newTokenAddress, newTokenId, 1);

  // update the delegatee's voting power based on new ERC1155 nft's multiplier
  _syncVotingPower(msg.sender, registration);
+  _lockNft(msg.sender, newTokenAddress, newTokenId, 1);
}
```

<details>
<summary>Instances (1)</summary>

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L326-L329C25

</details>
