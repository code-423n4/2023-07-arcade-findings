| | issue |
| ----------- | ----------- |
| 1 | [expressions for constant values such as a call to keccak256(), should use immutable rather than constant](#1-expressions-for-constant-values-such-as-a-call-to-keccak256-should-use-immutable-rather-than-constant) |
| 2 | [state variables should be cached in stack variables rather than re-reading them from storage](#2-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-ones-not-caught-by-c4udit) |
| 3 | [Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement](#3-add-unchecked--for-subtractions-where-the-operands-cannot-underflow-because-of-a-previous-require-or-if-statement) |
| 4 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#4-x--y-costs-more-gas-than-x--x--y-for-state-variables) |
| 5 | [can make the variable outside the loop to save gas](#5-can-make-the-variable-outside-the-loop-to-save-gas) |
| 6 | [using > 0 costs more gas than != 0 when used on a uint in a require() statement](#6-using--0-costs-more-gas-than--0-when-used-on-a-uint-in-a-require-statement) |
| 7 | [ Ternary over if ... else](#7-ternary-over-if--else) |
| 8 | [should use arguments instead of state variable](#8-should-use-arguments-instead-of-state-variable) |
| 9 | [before some functions we should check some variables for possible gas save](#9-before-some-functions-we-should-check-some-variables-for-possible-gas-save) |
| 10 | [Use hardcoded address instead address(this)](#10-use-hardcoded-address-instead-addressthis) |
| 11 | [The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement](#11-the-use-of-a-logical-and-in-place-of-double-if-is-slightly-less-gas-efficient-in-instances-where-there-isnt-a-corresponding-else-statement-for-the-given-if-statement) |
| 12 | [Divisions which do not divide by -X cannot overflow or overflow so such operations can be unchecked to save gas](#12-divisions-which-do-not-divide-by--x-cannot-overflow-or-overflow-so-such-operations-can-be-unchecked-to-save-gas) |
| 13 | [Unbounded Gas Consumption Risk Due to External Call Recipients](#13-unbounded-gas-consumption-risk-due-to-external-call-recipients) |
| 14 | [Empty receive functions can cause gas issues](#14-empty-receive-functions-can-cause-gas-issues) |


## 1. expressions for constant values such as a call to keccak256(), should use immutable rather than constant

- [ReputationBadge.sol#L44](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L44)
- [ReputationBadge.sol#L45](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L45)
- [ReputationBadge.sol#L46](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L46)

- [ArcadeTreasury.sol#L48](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L48)
- [ArcadeTreasury.sol#L49](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L49)
- [ArcadeTreasury.sol#L50](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L50)


## 2. state variables should be cached in stack variables rather than re-reading them from storage (ones not caught by c4udit)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`lastAllowanceSet[token]` can be cached before #L308 for a possible reduce of a complex SLOAD if the condition in that line is true but even if it is false we only lose 3 gas which is way less than a gas cost of complex SLOAD. (its better to cache `lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN` instead because we reduce a summation operation use as well)
- [ArcadeTreasury.sol#L308](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L308)

`amountClaimed[recipient][tokenId]` can be cached before #L111 to reduce a complex SLOAD. for this instead of using += in #L116 we should use equals and use the cached version of the state var. (its better to cache `amountClaimed[recipient][tokenId] + amount` instead because we reduce a summation operation use as well)
- [ReputationBadge.sol#L111](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L111)

`baseURI` can be cached to save gas if `baseURI` length isnt 0. even if its 0 we only lose little gas for using this method but usually we save much more
- [BadgeDescriptor.sol#L49](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49)

`mintingAllowedAfter` can be cached before #L146 to possibly save 100 gas. if the condition is true we save 100 gas but even if its false we only lose 3 gas for using this method.
- [ArcadeToken.sol#L146](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146)


## 3. Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
this will stop the check for overflow and underflow so it will save gas

`block.number - grant.cliff` can not underfolw because its checked before
- [ARCDVestingVault.sol#L328](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L328)

also if we reach #L329 so we know from the checks before that line that `grant.cliff <= block.number < grant.expiration` so `grant.expiration - grant.cliff` can not underflow as well
- [ARCDVestingVault.sol#L329](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L329)


## 4. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas (13 or 113 each dependant on the usage see [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8))

`amountClaimed[recipient][tokenId]` shouldnt use +=. (also explained deeper in finding #6)
- [ReputationBadge.sol#L116](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L116)


## 5. can make the variable outside the loop to save gas

make the variable outside the loop and only give the value to variable inside

`success`
- [ArcadeTreasury.sol#L341](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341)

`registration`
- [NFTBoostVault.sol#L346](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L346)


## 6. using > 0 costs more gas than != 0 when used on a uint in a require() statement

This change saves 6 gas per instance. The optimization works until solidity version 0.8.13 where there is a regression in gas costs.

- [BadgeDescriptor.sol#L49](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49)


## 7. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [ArcadeTreasury.sol#L365-L370](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L365-L370)

- [NFTBoostVault.sol#L591-L593](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L591-L593)


## 8. should use arguments instead of state variable

This will save 100 gas because it gets rid of a storage read and instead uses a argument that we already have which gives the same result

use `_newMinter` instead of `minter` to reduce one SLOAD
- [ArcadeToken.sol#L136](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136)


## 9. before some functions we should check some variables for possible gas save

before transfer we should check for amount being 0 so the function doesnt run when its not gonna do anything

- [ArcadeTreasury.sol#L367](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L367)
- [ArcadeTreasury.sol#L369](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L369)

- [ARCDVestingVault.sol#L201](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L201)
- [ARCDVestingVault.sol#L217](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L217)

- [NFTBoostVault.sol#L657](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L657)

- [ReputationBadge.sol#L171](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L171)


## 10. Use hardcoded address instead address(this)

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this. - [Reference](https://twitter.com/transmissions11/status/1518507047943245824)

- [ARCDVestingVault.sol#L201](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L201)

- [NFTBoostVault.sol#L557](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L557)
- [NFTBoostVault.sol#L657](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L657)
- [NFTBoostVault.sol#L674](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L674)

- [ReputationBadge.sol#L167](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L167)

- [ArcadeAirdrop.sol#L66](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L66)


## 11. The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement

Using a double if statement instead of logical AND (&&) can provide similar short-circuiting behavior whereas double if is slightly more efficient.

- [NFTBoostVault.sol#L247](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L247)
- [NFTBoostVault.sol#L317](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L317)
- [NFTBoostVault.sol#L472](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L472)
- [NFTBoostVault.sol#L632](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L632)
- [NFTBoostVault.sol#L659](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L659)


## 12. Divisions which do not divide by -X cannot overflow or overflow so such operations can be unchecked to save gas

Make such found divisions are unchecked when ensured it is safe to do so

`totalBlocksPostCliff` is a positive number because we know `grant.cliff <= block.number < grant.expiration` from the checks before
- [ARCDVestingVault.sol#L330](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L330)

- [NFTBoostVault.sol#L494](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L494)
- [NFTBoostVault.sol#L633](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L633)

- [ArcadeToken.sol#L154](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L154)


## 13. Unbounded Gas Consumption Risk Due to External Call Recipients

In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using `addr.call`{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

- [ArcadeTreasury.sol#L341](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341)


## 14. Empty receive functions can cause gas issues

In Solidity, functions that receive Ether without corresponding functionality to utilize or withdraw these funds can inadvertently lead to a permanent loss of value. This is because if someone sends Ether to the contract, they may be unable to retrieve it. To avoid this, functions receiving Ether should be accompanied by additional methods that process or allow the withdrawal of these funds.

- [ArcadeTreasury.sol#L397](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L397)
