# QA Report

## Low Findings

### L01 - The merkle root wont revert if is set to `0x00`

If merkle root is set to `0x00` could leave the system in an insecure state where system will accept any message that it has never seen before and process it as if it were genuine.

This has happen before in the [nomad bridge incident](https://medium.com/numen-cyber-labs/nomadic-bridge-attack-incident-analysis-94716b93fa61).

> The Nomad team initialized the trusted root to be 0x00. To be clear, using zero values as initialization values is a common practice.

Instances:
- [ArcadeMerkleRewards.sol#L62](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/libraries/ArcadeMerkleRewards.sol#L62)
- [MerkleRewards.sol#L28](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/external/council/libraries/MerkleRewards.sol#L28)
- [ArcadeAirdrop.sol#L76](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/token/ArcadeAirdrop.sol#L76)

### L02 - `delegate()` can be called with no voting power

The `delegate()` functions in `ARCDVestingVault` and `NFTBoostVault` can be called by anyone, despite them not having any voting power.

This leads to changes in the storage variables, like pushing empty entries to the `votingPower`, and setting a `delegatee`, despite not delegating anything. 

As it will set a `registration.delegatee`, it will bypass the validations `if (registration.delegatee == address(0)) revert NBV_NoRegistration();` on `addTokens()`, and `updateNft()`. Making it possible to call those functions without "initializing" the corresponding registration storage via `addNftAndDelegate()`.

This can lead to unexpected effects.

Instances:

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L182-L212
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L260-L282

#### Recommended Mitigation Steps

Verify that `_currentVotingPower(registration) > 0`

### L03 - `pure` functions should be reading or writing memory

From [solidity lang pure functions](https://docs.soliditylang.org/en/v0.8.21/contracts.html#pure-functions)
> Pure functions promise not to read from or modify the state.

Instances: 
- [BaseVotingVault.sol#L179-L182](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/BaseVotingVault.sol#L179-L182)
- [Storage.sol#L30-L31](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/external/council/libraries/Storage.sol#L30-L31)
- [Storage.sol#L65-L66](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/external/council/libraries/Storage.sol#L65-L66)
- [Storage.sol#L94-L95](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/external/council/libraries/Storage.sol#L94-L95)
- [Storage.sol#L109-L110](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/external/council/libraries/Storage.sol#L109-L110)
- [Storage.sol#L145-L146](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/external/council/libraries/Storage.sol#L145-L146)
- [ARCDVestingVaultStorage.sol#L49](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/libraries/ARCDVestingVaultStorage.sol#L49)
- [NFTBoostVaultStorage.sol#L74](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/libraries/NFTBoostVaultStorage.sol#L74)


### L04 - Use a two-step transfer function for the Minter role in `ArcadeToken`

The `ArcadeToken` has a very important Minter role which can be changed via a `setMinter()` function.

In order to prevent the role from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), require that the recipient of the new minter role actively accept via a contract call of its own.

Instances:
- [ArcadeToken.sol#L132-L137](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L132-L137)

### L05 - `NFTBoostVault::updateNft()` allows to deposit any NFT

`addNftAndDelegate` reverts when depositing any random NFT because of the `if (multiplier == 0) revert NBV_NoMultiplierSet();` in `_registerAndDelegate()`.

But `updateNft()` does not have that check, and allows anyone to update and deposit any NFT, leading to a multiplier of 0, and thus a voting power of 0, despite the user having deposited any other tokens.

This might also be combined with other issues for a bigger severity issue. We recommend that the same check to revert on `multiplier == 0` is applied on `updateNft()`.

Instances:
- [NFTBoostVault.sol#L305-L330](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L305-L330)
- [NFTBoostVault.sol#L477](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L477)

### L06 - If [`token`](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/ARCDVestingVault.sol#L61) has hooks the grant owner can revert when admin try to revoke role

A evil granted owner can avoid being [`revoke`](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/ARCDVestingVault.sol#L157), he can revert triggering a **DOS** to avoid manager to revoke role.

### L07 - If [`token`](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/ARCDVestingVault.sol#L61) has hooks the manager could **reenter** to `revokeGrant` draining the contract

A token with hook could leave a reentrancy issue, letting manager to draind funds.

### L08 - The storage slots/hashes have known pre-images.

Knowledge of pre-images might allow manipulation of specific mapping parameters by crafting special keys. This is possible if a mapping parameter's storage slot lands on a specific slot.

It might be best to decrement them by one so that finding the pre-image would be harder.

Instances:
- [Storage.sol](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/external/council/libraries/Storage.sol)

Example:
```solidity
bytes32 typehash = keccak256("mapping(address => AddressUint)");
bytes32 offset = keccak256(abi.encodePacked(typehash, name));
assembly {
    data.slot := sub(offset, 1) // keccak256(NAME) - 1, to avoid preimage attack
}
```

Also please review this [eip-1967](https://eips.ethereum.org/EIPS/eip-1967#security-considerations)

### L09 - Type of `tokenId` in `ReputationBadge` and `NftBoostVault` do not match, and can lead to tokens not being usable as multipliers

`ReputationBadge` tokens are meant to be used as multipliers of voting power in `NftBoostVault`.

Users can mint badges with `tokenId` up to `uint256`, but `NftBoostVault` only allows to set `tokenId` up to `uint128` to work as multipliers.

Tokens with `tokenId > type(uint128).max` won't be able to be used as multipliers.

`tokenId` in `ReputationBadge` (uint256):

```solidity
    function mint(
        address recipient,
@>      uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L100

`tokenId` in `NftBoostVault` (uint128):

```solidity
@>  function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L363

Consider setting the same `tokenId` type for the expected NFTs to be used in the contract.

### L10 - You can bypass the ERC1155 check and deposit anything if `tokenId == 0`

The check on [NFTBoostVault.sol#L472](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/NFTBoostVault.sol#L472) can be easily bypass if the ERC1155 token id is `0`:
`if (_tokenAddress != address(0) && _tokenId != 0) {`

**POC:**
```solidity!
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;
import "forge-std/Test.sol";
import "../contracts/NFTBoostVault.sol";
import "../contracts/token/ArcadeToken.sol";

contract ExampleTest is Test {
    NFTBoostVault public boostVault;
    ArcadeToken public token;

    function setUp() public {
        vm.roll(10);
        token = new ArcadeToken(address(this), address(this));
        boostVault = new NFTBoostVault(IERC20(address(token)), 1, address(this), address(this));
    }

    function testDepositAnyERC1155() public {
        // create a fake erc1155 (could also happen with a )
        FakeERC1155 f = new FakeERC1155();

        token.approve(address(boostVault),100);
        boostVault.addNftAndDelegate(
            1,
            0, // uint128 tokenId,
            address(f), //address tokenAddress,
            address(0xdead) // delegate
        );

        token.transfer(address(0x02), 1);
        vm.prank(address(0x02));
        token.approve(address(boostVault),100);

        vm.prank(address(0x02));
        boostVault.addNftAndDelegate(
            1,
            0, // uint128 tokenId,
            address(f), //address tokenAddress,
            address(0xdead01)
        );

        vm.prank(address(0xdead));
        boostVault.delegate(address(0x01));

        vm.prank(address(0xdead01));
        boostVault.delegate(address(0x01));

    }
}

contract FakeERC1155 {
    fallback() external {}
}
```

### L11 - Approve zero first

Some ERC20 tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value. For example Tether (USDT)'s approve() function will revert if the current approval is not zero, to protect against front-running changes of approvals.

Instances:
- [ArcadeMerkleRewards.sol#L86](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/libraries/ArcadeMerkleRewards.sol#L86)

## Non-Critical Findings

### NC01 - `NFTBoostVault` sets an unused `initialized` value in storage

The `"initialized"` storage variable is set on the constructor, but not used by the `NFTBoostVault`, nor other contract, and can't be accessed by any public or external method.

Recommendation: Verify if it is actually needed, or remove it in case it is not.

```solidity
Storage.set(Storage.uint256Ptr("initialized"), 1); // @audit check
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L91

### NC02 - `ARCDVestingVault` inherits twice from `HashedStorageReentrancyBlock`

`ARCDVestingVault` inherits both from `HashedStorageReentrancyBlock`, and `BaseVotingVault`.

But `BaseVotingVault` is already inheriting from `HashedStorageReentrancyBlock`, as well

Remove the unncessary inheritance from `HashedStorageReentrancyBlock` on `ARCDVestingVault`.

Instances:
- [ARCDVestingVault.sol#L48](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L48)