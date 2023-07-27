# Findings Summary

| ID     | Title                                                        | Severity     |
| ------ | ------------------------------------------------------------ | ------------ |
| [N-01] | Magic number should be avoided                               | Non-Critical |
| [N-02] | NFTBoostVault does not support token whose tokenId is 0      | Non-Critical |
| [N-03] | onERC1155Received should revert when receive non-boost token | Non-Critical |

# Detailed Findings

# [N-01] Magic number should be avoided

## Description

```solidity
    /// @dev Precision of the multiplier.
    uint128 public constant MULTIPLIER_DENOMINATOR = 1e3;
    
    uint128 multiplier = 1e3;

    // if a user does not specify a ERC1155 nft, their multiplier is set to 1
    if (tokenAddress == address(0) || tokenId == 0) {
        return 1e3;
    }
```

The multiplier has defined constant `MULTIPLIER_DENOMINATOR` in NFTBoostVault, but uses magic number in code, which is in poor readability. And if `MULTIPLIER_DENOMINATOR` changes, the code also needs to change elsewhere.

## Recommendations

Use MULTIPLIER_DENOMINATOR uniformly.

# [N-02] NFTBoostVault does not support NFT whose tokenId is 0

## Description

```solidity
    function getMultiplier(address tokenAddress, uint128 tokenId) public view override returns (uint128) {
        NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];

        // if a user does not specify a ERC1155 nft, their multiplier is set to 1
        if (tokenAddress == address(0) || tokenId == 0) {
            return 1e3;
        }

        return multiplierData.multiplier;
    }
```

The ERC1155 standard does not specify that the tokenId cannot be 0. If `tokenId == 0`, `getMultiplier` return `1e3` directly, even if the multiplier of `tokenId == 0` is later set to a different value, which makes the protocol incompatible with the type of tokenId 0.

```solidity
    function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
        uint256 mintPrice = mintPrices[tokenId] * amount;
        uint48 claimExpiration = claimExpirations[tokenId];

        if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));
        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
        if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
            revert RB_InvalidClaimAmount(amount, totalClaimable);
        }

        // increment amount claimed
        amountClaimed[recipient][tokenId] += amount;

        // mint to recipient
        _mint(recipient, tokenId, amount, "");
    }
```

ReputationBadge implements ERC1155 and does not restrict tokenId is not 0.

## Recommendations

Verify only tokenAddress, not tokenId

# [N-03] onERC1155Received should revert when receive non-boost token

## Description

```solidity
    function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {
        return this.onERC1155Received.selector;
    }
```

NFTBoostVault accept any ERC1155 tokens currently, which can result in tokens being accidentally stuck in contracts, and should be checked here to see if the token is a boost token.

## Recommendations

Check if the token is a boost token.