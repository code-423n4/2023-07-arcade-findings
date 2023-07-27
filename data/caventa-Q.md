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

5.

Update nft is able to replace old tokenID with same tokenID

See NFTBoostVault#updateNft

```solidity
 function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {
        if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);

        if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();

        NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

        // If the registration does not have a delegatee, revert because the Registration
        // is not initialized
        if (registration.delegatee == address(0)) revert NBV_NoRegistration();

        // if the user already has an ERC1155 registered, withdraw it
        if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
            // withdraw the current ERC1155 from the registration
            _withdrawNft();
        }

        // set the new ERC1155 values in the registration and lock the new ERC1155
        registration.tokenAddress = newTokenAddress;
        registration.tokenId = newTokenId;

        _lockNft(msg.sender, newTokenAddress, newTokenId, 1);

        // update the delegatee's voting power based on new ERC1155 nft's multiplier
        _syncVotingPower(msg.sender, registration);
    }
```

Although there is no severity issue if somebody does so, it should be prevented

Recommendation

Ensure if registration.tokenAddress == newTokenAddress, make sure registration.tokenId != newTokenId

5.

See ARCDVestingVault#addGrantAndDelegate

```solidity
if (cliff >= expiration || cliff < startTime) revert AVV_InvalidSchedule();
```

should be changed to 

```solidity
if (cliff >= expiration || cliff <= startTime) revert AVV_InvalidSchedule();
```

Because

cliff should not equal to startTime