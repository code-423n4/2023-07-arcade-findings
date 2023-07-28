# Gas Optimizations

### [G-01] To save gas on invalid input, the checks in `ReputationBadge.mint` can be reordered.

```diff
function mint(
  address recipient,
  uint256 tokenId,
  uint256 amount,
  uint256 totalClaimable,
  bytes32[] calldata merkleProof
) external payable {
-  uint256 mintPrice = mintPrices[tokenId] * amount;
  uint48 claimExpiration = claimExpirations[tokenId];
  if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));

+  uint256 mintPrice = mintPrices[tokenId] * amount;
  if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
  ...
}
```
