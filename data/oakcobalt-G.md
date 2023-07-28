See the markdown file with the details of this report [here](https://github.com/code-423n4/2023-07-arcade-findings/blob/main/data/oakcobalt-G.md).

### 1. Wasteful repeated storage writing within one transaction
In NFTBoostVault.sol - `updateNft()`, a user can change their Nft and sync their voting power based on the new nft. 

However, in this function `_syncVotingPower()` is performed twice. The first time `_withdrawNft()` will set `registration.tokenAddress` and `registration.tokenId` to 0, and then  `_syncVotingPower()` will update `registration.lastVotingPower`. The second time, `_syncVotingpower()` is called at the end of `updateNft()` where new `regsitation.tokenAddress`, and new `registion.tokenId` is written into storage.

Because registration info is first written to 0 and again to the new token info with new voting power, wasteful gas operations are performed.

```solidity
    function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {
...
        if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
            // withdraw the current ERC1155 from the registration
 |>         _withdrawNft();
        }
        // set the new ERC1155 values in the registration and lock the new ERC1155
        registration.tokenAddress = newTokenAddress;
        registration.tokenId = newTokenId;

        _lockNft(msg.sender, newTokenAddress, newTokenId, 1);

        // update the delegatee's voting power based on new ERC1155 nft's multiplier
 |>     _syncVotingPower(msg.sender, registration);
```
```solidity
    function _withdrawNft() internal {
        // load the registration
        NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];
...
        // remove ERC1155 values from registration struct
|>      registration.tokenAddress = address(0);
|>      registration.tokenId = 0;

        // update the delegatee's voting power based on multiplier removal
|>      _syncVotingPower(msg.sender, registration);
}
```
```solidity
    function _syncVotingPower(address who, NFTBoostVaultStorage.Registration storage registration) internal {
...
            votingPower.push(registration.delegatee, delegateeVotes + uint256(change));
...
        registration.latestVotingPower = uint128(newVotingPower);
...
```
Recommendation:

It’s recommended to only update `_syncVotingPower()` at the end of the transaction. And do not need to execute the full `_withdrawNft()` function, instead only need to transfer the ERC1155 back to user.