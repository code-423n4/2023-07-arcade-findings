1. time variables use uint32 instead of uint256

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L97

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L55



2.Pack `minter` and `mintingAllowedAfter` info one slot to save more gas.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L81#L94

If we replace uint256 with uint32 and we can packed `minter` and `mintingAllowedAfter` into one slot to save more gas.

```shell
slot
0
type: <inherited contract>.variable (bytes)
unallocated (8)
uint32: mintingAllowedAfter (4)
address: minter (20)
```

3.Use calldata instead of memory to save more gas.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L692#L699

```solidity
- function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {
+ function onERC1155Received(address, address, uint256, uint256, bytes calldata) public virtual returns (bytes4) {
     return this.onERC1155Received.selector;
 }

```

4.<array>.length should not be looked up in every loop of a for-loops

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144

5.use uncheck{++i} instead of i++

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144