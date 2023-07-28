## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Using a positive conditional flow to save a NOT opcode | 2 | - |
| [G-02] | Using storage instead of memory for structs/arrays saves gas | 2 | - |
| [G-03] | Use != 0 instead of > 0 for unsigned integer comparison | 2 | - |
| [G-04] | Avoid emitting storage values | 7 | - |
| [G-05] | Amounts should be checked for 0 before calling a transfer | 5 | - |
| [G-06] | Multiple accesses of a mapping/array should use a local variable cache | 2 | - |


## Gas Optimizations  

## [G-1] Using a positive conditional flow to save a NOT opcode

Estimated savings: 3 gas

```solidity
file: /contracts/nft/ReputationBadge.sol

343        if (!success) revert T_CallFailed();
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L343

```solidity
file: /contracts/nft/ReputationBadge.sol

110        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L110

## [G-2] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
file: /contracts/NFTBoostVault.sol

609        NFTBoostVaultStorage.Registration memory registration

628        NFTBoostVaultStorage.Registration memory registration
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L609

## [G-3] Use != 0 instead of > 0 for unsigned integer comparison

it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity
file: /contracts/NFTBoostVault.sol

589        if (change > 0) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L589

```solidity
file: /contracts/nft/BadgeDescriptor.sol

49      return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49


## [G-4] Avoid emitting storage values

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

```solidity
file: /contracts/token/ArcadeToken.sol

136        emit MinterUpdated(minter);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136

```solidity
file: /contracts/token/ArcadeTokenDistributor.sol

81        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);

97        emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);

113       emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);

129       emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);

146       emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);

163       emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L81


## [G-5] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

```solidity
file: /contracts/NFTBoostVault.sol

258        token.safeTransfer(msg.sender, amount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L258

```solidity
file: /contracts/nft/ReputationBadge.sol#

171         payable(recipient).transfer(balance);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L171

```solidity
file: /contracts/token/ArcadeAirdrop.sol

67        token.safeTransfer(destination, unclaimed);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L67

```solidity
file: /contracts/ARCDVestingVault.sol

167        token.safeTransfer(who, withdrawable);

172        token.safeTransfer(msg.sender, remaining);

201        token.transferFrom(msg.sender, address(this), amount);

217        token.safeTransfer(recipient, amount);

252        token.safeTransfer(msg.sender, withdrawable);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L167

```solidity
file: /contracts/token/ArcadeTokenDistributor.sol

79        arcadeToken.safeTransfer(_treasury, treasuryAmount);

95        arcadeToken.safeTransfer(_devPartner, devPartnerAmount);

111       arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);

127       arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);

144       arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);

161       arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L79

## [G‑6] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

```solidity
file: /contracts/ArcadeTreasury.sol

308        if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {

309        revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L308-L309

