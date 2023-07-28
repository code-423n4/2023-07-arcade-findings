## [L-01] Phony NFTs can be registered

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L305
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L462

The only check against phony NFTs the NFTBoostVault implements is checking whether the user owns the said NFT and whether the NFT's id is 0 or not.

This allows users to register phony NFTs as valid badges in the vault.

```solidity
if (_tokenAddress != address(0) && _tokenId != 0) {
  if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();

	multiplier = getMultiplier(_tokenAddress, _tokenId);

	if (multiplier == 0) revert NBV_NoMultiplierSet();
}
```

## Recommendation

Consider adding a check for whether an NFT actually has a multiplier.

```solidity
if(_getMultipliers()[tokenAddress][tokenId] == 0) revert();
```