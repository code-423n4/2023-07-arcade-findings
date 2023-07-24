## Summary

These are the findings for Low/Non-Critical vulnerabilities. 

## Vulnerability Details
### [L001] - Unsafe ERC20 Operation(s)

../2023-07-arcade/contracts/ARCDVestingVault.sol::201 => token.transferFrom(msg.sender, address(this), amount);
../2023-07-arcade/contracts/ArcadeTreasury.sol::367 => payable(destination).transfer(amount);
../2023-07-arcade/contracts/ArcadeTreasury.sol::391 => IERC20(token).approve(spender, amount);
../2023-07-arcade/contracts/NFTBoostVault.sol::657 => token.transferFrom(from, address(this), amount);
../2023-07-arcade/contracts/libraries/ArcadeMerkleRewards.sol::86 => token.approve(address(votingVault), uint256(totalGrant));
../2023-07-arcade/contracts/nft/ReputationBadge.sol::171 => payable(recipient).transfer(balance);

## Impact

ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least wrap each operation in a required statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.

## [G003] Mitigations
```
Poor Quality
IERC20(token).transferFrom(msg.sender, address(this), amount);
```
```
Corrected codes
import {SafeERC20} from "openzeppelin/token/utils/SafeERC20.sol";

IERC20(token).safeTransferFrom(msg.sender, address(this), amount);
```
```
Corrected codes
bool success = IERC20(token).transferFrom(msg.sender, address(this), amount);
require(success, "ERC20 transfer failed");
```

## Vulnerability Details
### L005 - Do not use Deprecated Library Functions

../2023-07-arcade/contracts/ArcadeTreasury.sol::90 => _setupRole(ADMIN_ROLE, _timelock);
../2023-07-arcade/contracts/nft/ReputationBadge.sol::79 => _setupRole(ADMIN_ROLE, _owner);

## Impact

The usage of deprecated library functions should be discouraged. This issue is mostly related to OpenZeppelin libraries. The _setupRole function has been deprecated in the OpenZeppelin libraries. This function was used to set up initial roles for a system during the constructor phase of a contract. However, as of recent updates, the _setupRole method has been replaced with the _grantRole function.

The _grantRole function performs the same operations as _setupRole, but it includes additional checks on the calling account. This change was made to discourage the usage of _setupRole outside of the constructor, which would effectively circumvent the admin system imposed by AccessControl github.com.

## [G003] Mitigations
Use the `_grantRole` function. ADMIN_ROLE is granted to the admin account during the contract's construction. This is similar to how _setupRole would be used, but it includes the additional access control checks provided by _grantRole.
