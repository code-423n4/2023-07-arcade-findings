# QA Report

## Summary

| id | title |
| --- | --- |
| [L-01](#l-01-delegate-doesnt-check-if-registration-exists) | `delegate` doesn't check if `registration` exists |
| [L-02](#l-02-thresholds-in-arcadetreasury-cannot-be-completely-removed) | thresholds in `ArcadeTreasury` cannot be completely removed |
| [L-03](#l-03-nftboostingvault-cant-use-erc1155-with-tokeniduint128max) | `NFTBoostingVault` can't use ERC1155 with `tokenId>uint128.max` |
| [L-04](#l-04-reputationbadge-is-erc1155burnable) | `ReputationBadge` is `ERC1155Burnable` |
| [R-01](#r-01-calculation-in-arcdvestingvault_syncvotingpower-can-be-simplified) | calculation in `ARCDVestingVault::_syncVotingPower` can be simplified |
| [R-02](#r-02-unnecessary-delegatee-check-when-claiming-airdrop) | unnecessary delegatee check when claiming airdrop |
| [R-03](#r-03-balance-read-long-before-it-is-used) | `balance` read long before it is used) |
| [R-04](#r-04-unnecessary-balanceof-check-for-nft-ownership) | unnecessary `balanceOf` check for NFT ownership  |
| [R-05](#r-05-unnecessary-equality-check-in-nftboostvault_getwithdrawableamount) | unnecessary equality check in `NFTBoostVault::_getWithdrawableAmount` | 
| [NC-01](#nc-01-addnftanddelegate-will-write-either-tokenaddress-or-tokenid-to-storage-is-a-user-just-sends-one-of-them) | `addNftAndDelegate` will write either `tokenAddress` or `tokenId` to storage is a user just sends one of them |

## Low

### L-01 `delegate` doesn't check if `registration` exists

When calling `NFTBoostVault::delegate` a user can delegate their votes to another account. The issue is however that the call doesn't check if the registration exists before writing the new `delegatee`

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L185

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L209
```solidity
File: NFTBoostVault.sol

185:        NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

            // no checks for registration existance

209:        registration.delegatee = to;
```

Thus a user can set a `delegatee` before registering. This would block their account from `addNftAndDelegate` uses `delegatee` to check the the registration exists:

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L488
```solidity
File: NFTBoostVault.sol

488:        if (registration.delegatee != address(0)) revert NBV_HasRegistration();
```

The user can however recover either with claiming an airdrop or `addTokens`. That would add an amount letting the user withdraw (or keep their votes).

The same bug exists in `ARCDVestingVault`.

#### Proof of Concept
Test in `test/NFTBoostVault.t.sol`:
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../contracts/NFTBoostVault.sol";
import "../contracts/token/ArcadeToken.sol";

contract NFTBoostVaultTest is Test {

    address private admin = address(this);
    ArcadeToken private token = new ArcadeToken(admin,admin);

    function testDelegateNonExistentRegistration() public {
        NFTBoostVault vault = new NFTBoostVault(token,0,admin,admin);
        
        // a user can delegate with a non existent registration
        vault.delegate(address(0x1111));

        // since the `delegatee` is then set they cannot register
        vm.expectRevert(NBV_HasRegistration.selector);
        vault.addNftAndDelegate(1000, 0, address(0), admin);
        
        // and since delegate checks for address(0) they cannot change back
        vm.expectRevert(abi.encodeWithSelector(NBV_ZeroAddress.selector,"to"));
        vault.delegate(address(0));

        vault.setAirdropContract(admin);
        token.approve(address(vault), 1000);

        // can be recovered with addTokens (which would allow withdraw resetting the account)
        vault.addTokens(1000);
    }
}
```

#### Recommendations
Consider checking if the registration exists in `delegate` (both `NFTBoostVault` and `ARCDVestingVault`).

### L-02 thresholds in `ArcadeTreasury` cannot be completely removed

When setting token spending thresholds in `ArcadeTreasury` a series of checks are done, that `small != 0` and that `small <= medium <= large`:
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L273-L278
```solidity
File: ArcadeTreasury.sol

273:        if (thresholds.small == 0) revert T_ZeroAmount();
274:
275:        // verify thresholds are ascending from small to large
276:        if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small) {
277:            revert T_ThresholdsNotAscending();
278:        }
```

This makes it so that a threshold cannot be completely removed. It can however be set to `1 wei` which is effectively removed.

#### Recommendation
Consider removing the check for `small == 0` so that a token threshold can be removed.

### L-03 `NFTBoostingVault` can't use ERC1155 with `tokenId>uint128.max`

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L116

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L305
```solidity
File: NFTBoostVault.sol

116:        uint128 tokenId,

305:    function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {
```

Only NFTs with `tokenId` that fits in `uint128` can be used. It is not uncommon to use the higher bits of the `uint256` to encode information in an NFT `tokenId`. This can make the tokenIds very "large" when considered numbers.

#### Recommendation
Consider using `uint256` for `tokenId` according to `ERC1155`


### L-04 `ReputationBadge` is `ERC1155Burnable`

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L39
```solidity
File: nft/ReputationBadge.sol

                                                       ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
39:contract ReputationBadge is ERC1155, AccessControl, ERC1155Burnable, IReputationBadge {
```

This could cause accidental unintentional burns of tokens. Could also be abused by user with allowances since  `ERC1155Burnable` allows that too.

#### Recommendation
Consider only inheriting only from `ERC1155` not `ERC1155Burnable`.

## Refactor

### R-01 calculation in `ARCDVestingVault::_syncVotingPower` can be simplified

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L350-L352
```solidity
File: ARCDVestingVault.sol

350:        int256 change = int256(newVotingPower) - int256(grant.latestVotingPower);
351:        // we multiply by -1 to avoid underflow when casting
352:        votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));
```

can be simplified without casts to:

```solidity
uint256 change = grant.lastVotingPower - newVotingPower;
votingPower.push(grant.delegatee, delegateeVotes - change);
```

That removes all casts and is safer since the `int256` casts can theoretically overflow. A similar change could be made in `NFTBoostVault::_syncVotingPower` but that makes the code very hard to read. Consider to remove the theoretical possibility of conversion overflows there as well though.

### R-02 unnecessary delegatee check when claiming airdrop

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L155-L156
```solidity
File: NFTBoostVault.sol

155:            // if user supplies new delegatee address revert
156:            if (delegatee != registration.delegatee) revert NBV_WrongDelegatee(delegatee, registration.delegatee);
```

This will only cause unwanted reverts. The only way to claim an airdrop for an already existing registration is providing the correct `delegatee`. If this check is removed. The `delegatee` sent will not matter (for existing registrations). Hence the user cannot make an error.

### R-03 `balance` read long before it is used

in `_registerAndDelegate` the field `balance` is loaded from storage long before it is used. This impedes on code readability as it confuses the reader why the field suddenly appears:

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L480-L481

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L504-L505
```solidity
File: NFTBoostVault.sol

480:        // load this contract's balance storage
481:        Storage.Uint256 storage balance = _balance();

            // `balance` not used for another 23 rows

504:        // update this contract's balance
505:        balance.data += _amount;
```

Consider reading `balance` right before using it on row 505.

### R-04 unnecessary `balanceOf` check for NFT ownership

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L308

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L473
```solidity
File: NFTBoostVault.sol

308:        if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();

473:            if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();
```

This check is unnecessary because if `msg.sender` would not own the NFT the [transfer would fail](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L673-L675).

### R-05 unnecessary equality check in `NFTBoostVault::_getWithdrawableAmount`

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L611-L615
```solidity
File: NFTBoostVault.sol

611:        if (registration.withdrawn == registration.amount) {
612:            return 0;
613:        }
614:
615:        return registration.amount - registration.withdrawn;
```

The check `registration.withdrawn == registration.amount` is unnecessary as `registration.amount - registration.withdrawn` will return `0` when they are equal.

## Non critical/Informational

### NC-01 `addNftAndDelegate` will write either `tokenAddress` or `tokenId` to storage is a user just sends one of them

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L472-L478

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L500-L501
```solidity
File: NFTBoostVault.sol

472:        if (_tokenAddress != address(0) && _tokenId != 0) {
473:            if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();
474:
475:            multiplier = getMultiplier(_tokenAddress, _tokenId);
476:
477:            if (multiplier == 0) revert NBV_NoMultiplierSet();
478:        }

...

500:        registration.tokenId = _tokenId;
501:        registration.tokenAddress = _tokenAddress;
```

This doesn't really do anything because all other places also check `tokenAddress != address(0) && tokenId != 0`. It can however appear weird when queried. Consider not writing any of them of only one is passed.