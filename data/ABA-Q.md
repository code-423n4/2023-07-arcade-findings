# QA Report for contest Arcade.xyz

## Overview
During the audit, 7 low, 4 non-critical and 3 refactoring issues were found.

### Low Risk Issues

Total: 7 instances over 7 issues

|#|Issue|Instances|
|-|:-|:-:|
| [L-01] | `NFTBoostVault` registration token address and tokenId can each be set to arbitrary values (but not at the same time) | 1 |
| [L-02] | Booster NFT multiplier can be set bellow **1e3** | 1 |
| [L-03] | Spending amounts limits in `ArcadeTreasury` can be equal | 1 |
| [L-04] | `ARCDVestingVault::delegate` allows delegation to 0 address | 1 |
| [L-05] | `ARCDVestingVault::delegate` allows delegation without a grant existing | 1 |
| [L-06] | `NFTBoostVault::withdraw` does not check for valid registration | 1 |
| [L-07] | `NFTBoostVault::withdraw` does not properly clear registration if full staked tokens are withdrawn | 1 |

### Refactoring Issues

Total: 5 instances over 3 issues

|#|Issue|Instances|
|-|:-|:-:|
| [R-01]| Do not revert when exceeding the maximum mint cap in `ArcadeToken`; mint the maximum instead | 1 |
| [R-02]| Redundant checks in `ARCDVestingVault::claim` can be eliminated | 1 |
| [R-03]| `grant.created` is not used anywhere and can be removed from `ARCDVestingVault` logic | 3 |

### Non-critical Issues

Total: 3 instances over 4 issues

|#|Issue|Instances|
|-|:-|:-:|
| [NC-01]| Both GSC and normal governance transaction limits are using the same `blockExpenditure` limit check | 1 |
| [NC-02]| All `ADMIN_ROLE` roles can remove themselves BadgeDescriptor leaving the contract without an admin | 1 |
| [NC-02]| Incorrect, incomplete or misleading documentation | 2 |

#
## Low Risk Issues (7)
#

### [L-01] NFTBoostVault registration token address and tokenId can each be set to arbitrary values (but not at the same time)
##### Description

When registring to the `NFTBoostVault` via `addNftAndDelegate`, among others, tokenId and tokenAddress are user provided.

[If, however, both of these are not equal to 0 at the same time](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L472-L478), then the non-zero one will be saved to the registration entry without any further check

```Solidity
        if (_tokenAddress != address(0) && _tokenId != 0) {
            if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();


            multiplier = getMultiplier(_tokenAddress, _tokenId);


            if (multiplier == 0) revert NBV_NoMultiplierSet();
        }

        // ...

        registration.tokenId = _tokenId;
        registration.tokenAddress = _tokenAddress;

        // ...
```

Having a registration entry with a random `tokenAddress` or `tokenId` breaks protocol invariant and may lead to exploits in the future, but, as it is, it does not impact current operations since all code logic updates both at the same time

##### Instances (1)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L472-L478

##### Recommendation

Add an else clause that sets them both to 0 if not validated

```diff
diff --git a/contracts/NFTBoostVault.sol b/contracts/NFTBoostVault.sol
index 5f907ee..701c425 100644
--- a/contracts/NFTBoostVault.sol
+++ b/contracts/NFTBoostVault.sol
@@ -475,6 +475,10 @@ contract NFTBoostVault is INFTBoostVault, BaseVotingVault {
             multiplier = getMultiplier(_tokenAddress, _tokenId);
 
             if (multiplier == 0) revert NBV_NoMultiplierSet();
+        } else {
+            // cleans up if one of them was given with a set value
+            _tokenAddress = address(0);
+            _tokenId = 0;
         }
 
         // load this contract's balance storage

```

#

### [L-02] Booster NFT multiplier can be set bellow **1e3**
##### Description

**1e3** in the protocol logic represents a multiplier by 1. [getMultiplier](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L422-L424) returns the default **1e3** or the set multiplier.

While there is a `MAX_MULTIPLIER` value that is checked when setting the multiplier via `setMultiplier`

```Solidity
    function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {
        if (multiplierValue > MAX_MULTIPLIER) revert NBV_MultiplierLimit();


        NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];
        // set multiplier value
        multiplierData.multiplier = multiplierValue;


        emit MultiplierSet(tokenAddress, tokenId, multiplierValue);
    }
```

there is not minimum set. As such going bellow **1e3** will result in less voting power then not having an NFT locked.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L493-L494

##### Instances (1)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L363-L371
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L493-L494

##### Recommendation

Add a minimum `multiplierValue` check in the `setMultiplier` function for **1e3**.

#

### [L-03] Spending amounts limits in `ArcadeTreasury` can be equal 
##### Description

When setting these spending limits via `ArcadeTreasury::setThreshold` they are incorrectly checked that are in ascending order:

```Solidity
        // verify thresholds are ascending from small to large
        if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small) {
            revert T_ThresholdsNotAscending();
        }
```
The corner case the check is allowing is where:
- small=medium=large
- small=medium < large
- small < medium=large


##### Instances (1)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L275-L278

##### Recommendation

Modify the if checks to be `<=` instead of `<` so that no equality is permitted

#

### [L-04] `ARCDVestingVault::delegate` allows delegation to 0 address
##### Description

There is no check in `ARCDVestingVault::delegate` for 0 address on the `to` input. As such a user may mistakenly delegate his voting power to the null address. Although not in the current project form, this may lead to a sever issue in the future.

##### Instances (1)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L260C14-L262

##### Recommendation

Add a check for 0 address on the `to` argument

#

### [L-05]  `ARCDVestingVault::delegate` allows delegation without a grant existing
##### Description

`ARCDVestingVault::delegate`'s purpose is to delegate the grant voting power to an address. The function unfortunately does not check if there actually is a grant associated with the caller. As such, a `grant` entry is created with only the delegate set to it.

This breaks protocol invariant and may lead to more critical issues in the future.

##### Instances (1)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L260-L262

##### Recommendation

Add a check that the grant is set, similar to the `claim` function:
` if (grant.allocation == 0) revert AVV_NoGrantSet();`

#

### [L-06] `NFTBoostVault::withdraw` does not check for valid registration
##### Description

When withdrawing via `NFTBoostVault::withdraw` the function does not check if there is a valid registration associated with the user. 
Although function does revert, because of the `if (withdrawable < amount) revert NBV_InsufficientWithdrawableBalance(withdrawable);` check,  
it should be first checked that a registration does exist

##### Instances (1)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L223-L236

##### Recommendation

Add the following check to the withdraw function:

```Solidity
        // If the registration does not have a delegatee, revert because the Registration
        // is not initialized
        if (registration.delegatee == address(0)) revert NBV_NoRegistration();
```

#

### [L-07] `NFTBoostVault::withdraw` does not properly clear registration if full staked tokens are withdrawn
##### Description

`NFTBoostVault::withdraw` deletes a registration if the full staked amount is withdrawn. Due to a fault check (that actually comes from a different issue) a registration may come with a token address but not a token ID (or vice-versa). As such, clearing the registration as the current code does leaves either the injected `tokenAddress` or the `tokenId`

```Solidity
    if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
        _withdrawNft();
    }
    // delete registration. tokenId and token address already set to 0 in _withdrawNft()
```

The comment is actually wrong and in the described case it does not clear.

##### Instances (1)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L247-L250

##### Recommendation

Clear tokenAddress and tokenId after the if regardless:

```diff
        if (registration.withdrawn == registration.amount) {
            if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
                _withdrawNft();
            }
-            // delete registration. tokenId and token address already set to 0 in _withdrawNft()
+            // delete registration
+            registration.tokenId = 0;
+            registration.tokenAddress = 0;
            registration.amount = 0;
            registration.latestVotingPower = 0;
```

#

#### Refactoring Issues (3)
#

### [R-01] No not revert when exceeding the maximum mint cap in `ArcadeToken`; mint the maximum instead
##### Description

If by mistake when the cooldown period has passed and minter attempts to mint. If he introduces a value above the allowed minted cap, currently it reverts. 

```Solidity
        // inflation cap enforcement - 2% of total supply
        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
        if (_amount > mintCapAmount) {
            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
        }
```
Since this case may happen out of miscalculation and the intent of the user would still be to mint as much tokens as possible, a suggest here is to cap the `amount` to `mintCapAmount` if larger.

##### Instances (1)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L153-L157

##### Recommendation

Cap the `amount` to `mintCapAmount` if larger and not revert.

#

### [R-02] Redundant checks in `ARCDVestingVault::claim` can be eliminated
##### Description

In `ARCDVestingVault::claim` there is a check done with regards to amount and withdrawable:

```Solidity
        // update the grant's withdrawn amount
        if (amount == withdrawable) {
            grant.withdrawn += uint128(withdrawable);
        } else {
            grant.withdrawn += uint128(amount);
            withdrawable = amount;
        }


        // update the user's voting power
        _syncVotingPower(msg.sender, grant);


        // transfer the available amount
        token.safeTransfer(msg.sender, withdrawable);
    }
```
This check is redundant and the subsequently following and can be simply replaced with:

```Solidity
        // update the grant's withdrawn amount
        grant.withdrawn += uint128(amount);

        // update the user's voting power
        _syncVotingPower(msg.sender, grant);


        // transfer the available amount
        token.safeTransfer(msg.sender, amount);
    }
```

##### Instances (1)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L240-L246

##### Recommendation

Replace the code with the above suggestion.

#

### [R-03] `grant.created` is not used anywhere and can be removed from `ARCDVestingVault` logic
##### Description

When creating a grant using `ARCDVestingVault::addGrantAndDelegate` an optional argument can be passed `startTime` which is:
> Optionally set a start time in the future. If set to zero then the start time will be made the block tx is in.

As it indicates, this value is [set to the current block number](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L104-L107) if 0 was provided:

```Solidity
    // if no custom start time is needed we use this block.
    if (startTime == 0) {
        startTime = uint128(block.number);
    }
```

However the value is not used anywhere after this point and can be safely removed (sponsor confirmed no usage is currently intended for it).

##### Instances (3)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L39
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L84-L107
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/libraries/ARCDVestingVaultStorage.sol#L33

##### Recommendation

Remove the variable from the code as well as the documentation

#

## Non-critical Issues (4)
#

### [NC-01] Both GSC and normal governance transaction limits are using the same `blockExpenditure` limit check
##### Description

Both GSC and normal governance transaction limits are using the same `blockExpenditure` limit check. As such, in the off change that both GSC and a normal governance transaction happens at the same time (block), and both reaching their amount limit, one will fail.

##### Instances (2)

GSC uses the same functions as the regular spends/approvals

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L119
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L200

##### Recommendation

#

### [NC-02] All `ADMIN_ROLE` roles can remove themselves BadgeDescriptor leaving the contract without an admin
##### Description

All `ADMIN_ROLE` roles can remove themselves BadgeDescriptor leaving the contract without an admin

##### Instances

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol

##### Recommendation

Override the revokeRole function to keep track of admins and not allow all to be removed

#

### [NC-03] Incorrect, incomplete or misleading documentation
##### Description

There are instances of incorrect, incomplete or misleading documentation in the project. 

##### Instances (2)

- in [ARCDVestingVault:addGrantAndDelegate](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L84-L88) _Timestamp_ and _time_ are referenced but code only uses `block.number`
    - change the natspec to reflect this
- [ArcadeTreasury::batchCalls](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L325-L332) should be better document with regards to the `T_InvalidTarget` limitation. 
    - add the extra explanation from [errors/Treasury::T_InvalidTarget](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/errors/Treasury.sol#L47-L51) to the function natpsec

##### Recommendation

Resolve the indicated issues

#
