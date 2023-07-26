1.

See AcradeToken#mint,

```solidity
     uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
        if (_amount > mintCapAmount) {
            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
        }
```

mintCapAmount should return 2% of totalSupply(). However, if the value of totalSupply() is too small mintCapAmount could be 0 which should not be allowed.

Recommendation: Add 

```solidity
require(mintCapAmount > 0, "Mint cap amount should not be 0");
```

to ensure mintCapAmount always greater than 0

2.

Minter is a normal human that would make silly mistake one day. If he mints a wrong token amount, he needs to wait for another year to mint another token amount.

See AcradeToken#mint,

```solidity
mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;
```

Recommendation

Instead of having 1 function for minting. Better seperate the minting function into two where the minter needs to mint + confirmMint in order mint a token amount.

This is good where if the minter found if he mints the wrong amount, he can recall the mint function again and only confirm the minting after calling the confirmMint function.

3.

See BadgeDescription#tokenURI

```solidity
abi.encodePacked(baseURI, tokenId.toString())
```

It is not recommend to encodePacked two strings as string is dynamic type

See the following comment in https://docs.soliditylang.org/en/v0.8.11/abi-spec.html

If you use keccak256(abi.encodePacked(a, b)) and both a and b are dynamic types, it is easy to craft collisions in the hash value by moving parts of a into b and vice-versa. More specifically, abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c"). If you use abi.encodePacked for signatures, authentication or data integrity, make sure to always use the same types and check that at most one of them is dynamic. Unless there is a compelling reason, abi.encode should be preferred.

Change

abi.encodePacked to abi.encode

4.

In NFTBoostVault.sol

System assumes IERC1155 token is unable to have tokenID = 0 and system does not allow user to add, update, withdraw tokenID = 0.

For add,

See NFTBoostVault#_registerAndDelegate

```solidity
if (_tokenAddress != address(0) && _tokenId != 0) {
```

For update,

See NFTBoostVault#updateNft

```solidity
    if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);
```

For withdraw,

See NFTBoostVault#withdraw

```solidity
if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
```

All these checking to ensure tokenId != 0 should be removed 