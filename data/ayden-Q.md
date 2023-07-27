1.Use a 2-step ownership transfer pattern

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L132#L137

2.Ensure the value of `multiplierValue` is bigger than `MULTIPLIER_DENOMINATOR`.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L363#L371

`multiplierValue` should bigger than `MULTIPLIER_DENOMINATOR` 1e3 or those who lock NFT will get less votingPower than those who don't lock NFT.

3.Unsafe casting from uint256 to uint28

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L208

4.`setToken` missing an event to record the newly set token address

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L174#L179

5.`publishRoots` should ensure tokenId>0
Based on the known conditions, the ERC1155 NFT with tokenId=0 cannot be added to the NFTBoostVault pool