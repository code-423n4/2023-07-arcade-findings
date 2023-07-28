
## [G-01]  x -= y costs more gas than x = x - y for state variables

198: ` gscAllowance[token] -= amount;`
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L198

141: `unassigned.data -= amount;`
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L141

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L166

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L171

## [G-02] Use calldata instead of memory for function arguments that do not get mutated
57:`function setBaseURI(string memory newBaseURI) external onlyOwner {`
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L57

48:` function tokenURI(uint256 tokenId) external view override returns (string memory) {`
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L48

## [G-03]  Avoiding Initialization of Loop Index If It Is 0 and Avoid unnecessary read of array length in for loops can save gas

339: `for (uint256 i = 0; i < targets.length; ++i) {`
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L339

345: `for (uint256 i = 0; i < userAddresses.length; ++i) {`
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

## [G-04 ]  The result of a function call should be cached
```
633: return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L633

## [G-05] Use solidity version 0.8.19 to gain some gas boost
>Upgrade to the latest solidity version 0.8.19 to get additional gas savings.
>See latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

`pragma solidity 0.8.18;`

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeGSCCoreVoting.sol#L3

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L3

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L3
