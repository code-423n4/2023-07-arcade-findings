# QA Report

## Low Risk Issues
| Count | Explanation |
|:--:|:-------|
| [L-01] | User can fail to get Voting power multiplier despite having `ReputationBadge` NFT |
| [L-02] | Risk of Funds getting stuck in `ArcadeTokenDistributor` |
| [L-03] | No defence against return data bombs in `batchCalls` |  
| [L-04] | Cooldown period for GSC allowance can be bypassed in case of token with multiple addresses |   
| [L-05] | `NFTBoostVault` can accept Random NFTs whose `multiplier` is not set |   
| [L-06] | Potential DoS in `claim` and `revokeGrant` function because of underflow if `staleBlockLag` is too low |   

| Total Low Risk Issues | 6 |
|:--:|:--:|

### [L-01] User can fail to get Voting power multiplier despite having `ReputationBadge` NFT

## Description

Users can claim their Reputation Badge NFT by providing merkle Proofs. But before claiming, Badge Manager is expected to add data related to the Root, expiration and prices which is going to be used during `mint` process.

```solidity
File: ReputationBadge.sol

  function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
      if (_claimData.length == 0) revert RB_NoClaimData();
      if (_claimData.length > 50) revert RB_ArrayTooLarge();

      for (uint256 i = 0; i < _claimData.length; i++) {
          // ----SNIP: expiration check------

          claimRoots[_claimData[i].tokenId] = _claimData[i].claimRoot;
          claimExpirations[_claimData[i].tokenId] = _claimData[i].claimExpiration;
          mintPrices[_claimData[i].tokenId] = _claimData[i].mintPrice;
      }
  }

```

Issue here: There is no Checks for `_claimData[i].tokenId != 0`.

In case Badge Manager uses `_claimData[i].tokenId = 0`, then:

1. User will pay the price to mint the NFT.
2. User will call `addNftAndDelegate` in `NFTBoostVault` and expect to get a multiplier to his/her voting power.
3. Further `addNftAndDelegate` calls 2 functions: `_registerAndDelegate` and `_lockTokens`.
4. User will fail to transfer the NFT and get the Voting Rights multiplier because of Line 659 in `NFTBoostVault`.

```solidity
File: NFTBoostVault.sol

      function _lockTokens(
          address from,
          uint256 amount,
          address tokenAddress,
          uint128 tokenId,
          uint128 nftAmount
      ) internal {
          token.transferFrom(from, address(this), amount);

659:      if (tokenAddress != address(0) && tokenId != 0) {
              _lockNft(from, tokenAddress, tokenId, nftAmount);
          }
      }

```

## Recommendation

Add a condition in `publishRoots` function of  `ReputationBadge` such that it reverts in case `_claimData[i].tokenId` is equal to `0`.

### [L-02] Risk of Funds getting stuck in `ArcadeTokenDistributor`

## Description

`ArcadeTokenDistributor` contract is used for the distribution of Arcade Tokens between various stakeholders of the protocol.

Few points about it:

1. `arcadeToken` is not set initially.
2. Every function such as `toDevPartner` can be called only once throughout it's lifetime.
3. For every Function, there is NO check of `arcadeToken != address(0)`.

```solidity
File: ArcadeTokenDistributor.sol

  function toDevPartner(address _devPartner) external onlyOwner {
      if (devPartnerSent) revert AT_AlreadySent();
      if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");

      devPartnerSent = true;

      //------SNIP: Transfer----//

      //------SNIP: Event----//
  }

  function setToken(IArcadeToken _arcadeToken) external onlyOwner {
      //------SNIP: Validation----//

      arcadeToken = _arcadeToken;
  }

```

So there exist a possible risk where:

1. Total initial funds which is equal to the sum of distribution through all function is deposited in contract.
2. Owner by mistake calls any one method before setting the `arcadeToken`.

In this case, the remaining tokens for that called function will be stuck in the contract forever.

### [L-03] No defence against return data bombs in `batchCalls`

## Description

`batchCalls` function is supposed to be called by the `ADMIN_ROLE`.

```solidity
File: ArcadeTreasury.sol

341:     (bool success, ) = targets[i].call(calldatas[i]);
342:     // revert if a single call fails
343:     if (!success) revert T_CallFailed();

```

Here `(bool success, )` is actually the same as writing `(bool success, bytes memory data)` which basically means that even though the `data` is omitted it doesn’t mean that the contract does not handle it. 

Actually, the way it works is the `bytes data` that was returned from the `target` will be copied to memory. Memory allocation becomes very costly if the payload is big, so this means that if a `target` implements a fallback function that returns a huge payload, then the `msg.sender` of the transaction, in our case the `ADMIN_ROLE`, will have to pay a huge amount of gas for copying this payload to memory.

## Recommendation

Use a low-level assembly `call` since it does not automatically copy return data to memory

```solidity
  bool success;
  assembly {
      success := call(gas(), targets[i], 0, add(calldatas[i], 0x20), mload(calldatas[i]), 0, 0)
  }
```

### [L-04] Cooldown period for GSC allowance can be bypassed in case of token with multiple addresses

## Description

There exist ERC20 tokens that have more than one address which you can check [Here](https://github.com/d-xo/weird-erc20#multiple-token-addresses).

As per Natspac of `setGSCAllowance` function:
> There is a cool down period of 7 days after this function has been called where it cannot be called again.

But In case of multiple address token, 

```solidity
File: ArcadeTreasury.sol

308:    if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) { 
309:        revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);
310:    }

```

The above check will bypass and allow to set GSC allowance again within the cooldown period.

### [L-05] `NFTBoostVault` can accept Random NFTs whose `multiplier` is not set

## Description

There are 2 ways for users to register and add NFTs to enhance their voting power by a `multiplier`.

1. Using `addNftAndDelegate` Function
2. Receiving Airdrop through `airdropReceive` and then adding NFT through `updateNft`.

For the First process, In `_registerAndDelegate`, there is a check shown with (@->) below where in case if their is no Multiplier Set corresponding to `_tokenAddress` and `_tokenId` then the function reverts.

```solidity
File: NFTBoostVault.sol

  function _registerAndDelegate(
        address user,
        uint128 _amount,
        uint128 _tokenId,
        address _tokenAddress,
        address _delegatee
    ) internal {
        uint128 multiplier = 1e3;

        // confirm that the user is a holder of the tokenId and that a multiplier is set for this token
        if (_tokenAddress != address(0) && _tokenId != 0) {
            if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();

            multiplier = getMultiplier(_tokenAddress, _tokenId);

@->         if (multiplier == 0) revert NBV_NoMultiplierSet();
        }

        //---SNIP: Continuation---//
    }

```

This makes sure that only Selected NFTs and their corresponding tokenIds can be locked in the contract.

But in case a user uses second step or after doing first step, updates the NFT, then there is no check to make sure that the NFT locked in the contract is there in Multiplier Set.

```solidity
File: NFTBoostVault.sol

  function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {
      if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);

      if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();

      NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

      //----SNIP: Continuation---//
  }

```

## Recommendation

Add the following 2 lines in `updateNft` function:

```solidity

  multiplier = getMultiplier(newTokenAddress, newTokenId); 
  if (multiplier == 0) revert NBV_NoMultiplierSet();

```

### [L-06] Potential DoS in `claim` and `revokeGrant` function because of underflow if `staleBlockLag` is too low

## Description

In case, value of `staleBlockLag` is very low, then anyone can potentially do a Griefing Attack where he/she calls `queryVotePower` to clear everything more than `staleBlockLag` into the past for a grant owner (Note: `queryVotePower` function has no access control and anyone can call it) which might lead to voting power becoming `0` in case grant owner's last voting power update was before `staleBlockLag` number of blocks.

Now, there will be difference between `delegateeVotes` and `change` in the below line.

```solidity
File: ARCDVestingVault.sol

344:    uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);

350:    int256 change = int256(newVotingPower) - int256(grant.latestVotingPower);
351:    // we multiply by -1 to avoid underflow when casting
352:    votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));

```

Here, `delegateeVotes` will be zero as user have removed every past data and change will be a positive number as `grant.latestVotingPower` is still as it is. So subtracting anything from `0` will lead to underflow.

So, The line at 352 will revert. This means `_syncVotingPower` will be permanently DoSed for the user which can lead to DoS in `claim` and `revokeGrant` function.

This means the fund for that grant will be stuck in the contract forever.