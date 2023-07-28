| Number | Optimization Details                                                                                                                                                   | Instances |
| :----: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Do not calculate constant variables                                                                                                                                   |     6     |
| [G-02] | x += y and x -= y are more expensive than x = x + y and x = x-y for state variables                                                                                                                                   |     3     |
| [G-03] | State variables should be cached in stack variables rather than re-reading them from storage                                                                                                                                   |     2     |
| [G-04] | Low level call can be optimized with assembly                                                                                                                                   |     1     |
| [G-05] | Avoid contract existence checks by using low level calls                                                                                                               |     6     |
| [G-06] | Add unchecked blocks for subtractions where the operands cannot underflow                                                                                                               |     4     |
| [G-07] | Use assembly to check for address(0)                                                                                                               |     6     |
| [G-08] | The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement |     5     |
| [G-09] | >= costs less gas than >                                                         |     3     |
| [G-10] | <array>.length should not be looked up in every loop of a for-loop                                                       |     1     |

Total 10 issues

## [G-01] Do not calculate constant variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.Assign direct simple constant value after calculating off chain

_Total 6 instances - 2 file:_

### Instance#1-3:

```solidity
File: contracts/ArcadeTreasury.sol
48:    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");
49:    bytes32 public constant GSC_CORE_VOTING_ROLE = keccak256("GSC_CORE_VOTING");
50:    bytes32 public constant CORE_VOTING_ROLE = keccak256("CORE_VOTING");
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L48C1-L50C73

### Instance#4-6:

```solidity
File: contracts/ArcadeTreasury.sol
44:    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");
45:   bytes32 public constant BADGE_MANAGER_ROLE = keccak256("BADGE_MANAGER");
46:   bytes32 public constant RESOURCE_MANAGER_ROLE = keccak256("RESOURCE_MANAGER");
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L44C1-L46C83

## [G-02] x += y and x -= y are more expensive than x = x + y and x = x-y for state variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.Assign direct simple constant value after calculating off chain

_Total 3 instances - 2 file:_

### Instance#1-2:

```solidity
File: contracts/ArcadeTreasury.sol
117: gscAllowance[token] -= amount;

198: gscAllowance[token] -= amount;
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L117C8-L117C39

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L198C9-L198C39

### Instance#3:

```solidity
File: contracts/nft/ReputationBadge.sol
116: amountClaimed[recipient][tokenId] += amount;

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L116C8-L116C53


## [G-03] State variables should be cached in stack variables rather than re-reading them from storage

Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

_Total 2 instances - 2 file:_

### Instance#1:

```solidity
File: contracts/ArcadeTreasury.sol
//@audit lastAllowanceSet[token] on line 308
309:revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L309

### Instance#2:

```solidity
File: contracts/nft/ReputationBadge.sol
//@audit amountClaimed[recipient][tokenId] on line 111
116: amountClaimed[recipient][tokenId] += amount;

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L309

## [G-04] Low level call can be optimized with assembly

returnData is copied to memory even if the variable is not utilized: the proper way to handle this is through a low level assembly call.

_Total 1 instances - 1 file:_

### Instance#1:

```solidity
File: contracts/ArcadeTreasury.sol
341:(bool success, ) = targets[i].call(calldatas[i]);
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341

## [G-05] Avoid contract existence checks by using low level calls
Prior to Solidity 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls.

In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

_Total 6 instances - 3 file:_

### Instance#1-3:

```solidity
File: contracts/ArcadeTreasury.sol
369:IERC20(token).safeTransfer(destination, amount);

391: IERC20(token).approve(spender, amount);

674: IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L369

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L391

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L674

### Instance#4-5:

```solidity
File: contracts/NFTBoostVault.sol
473:if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();

556: IERC1155(registration.tokenAddress).safeTransferFrom(
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L473C13-L473C97

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L556

### Instance#6:

```solidity
File: contracts/nft/ReputationBadge.sol
84: descriptor = IBadgeDescriptor(_descriptor);
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L84C9-L84C52

## [G-06] Add unchecked blocks for subtractions where the operands cannot underflow
There are some checks to avoid an underflow, so it's safe to use unchecked to have some gas savings.
_Total 4 instances - 2 file:_

### Instance#1-3:

```solidity
File: contracts/ARCDVestingVault.sol
// @audit check on line 213
215:  unassigned.data -= amount;

// @audit check on line 319
328:uint256 blocksElapsedSinceCliff = block.number - grant.cliff;

// @audit check on line 319 and 323
329:uint256 totalBlocksPostCliff = grant.expiration - grant.cliff;
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L215C8-L215C35

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L328C8-L328C70

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L329

### Instance#4:

```solidity
File: contracts/NFTBoostVault.sol
// @audit check on line 232
239:balance.data -= amount;
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L239C9-L239C32

## [G-07] Use assembly to check for address(0)
Using assembly for address comparisons in Solidity can save gas because it allows for more direct access to the Ethereum Virtual Machine (EVM), reducing the overhead of higher-level operations. Solidity's high-level abstraction simplifies coding but can introduce additional gas costs. Using assembly for simple operations like address comparisons can be more gas-efficient.

_Total 6 instances - 1 file:_

### Instance#1-6:

```solidity
File: contracts/NFTBoostVault.sol
247: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

317:if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

472:if (_tokenAddress != address(0) && _tokenId != 0) {

488: if (registration.delegatee != address(0)) revert NBV_HasRegistration();

632: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

659:if (tokenAddress != address(0) && tokenId != 0) {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L247C12-L247C88

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L317

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L472C9-L472C60

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L632

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L488

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L659

## [G-08] The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement

Using a double if statement instead of logical AND (&&) can provide similar short-circuiting behavior whereas double if is slightly more efficient.

_Total 5 instances - 1 file:_

### Instance#1-5:

```solidity
File: contracts/NFTBoostVault.sol
247: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

317: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

472: if (_tokenAddress != address(0) && _tokenId != 0) {

632: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

659:if (tokenAddress != address(0) && tokenId != 0) {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L247C12-L247C88

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L317

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L472C9-L472C60

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L632

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L659

## [G-09] >= costs less gas than >
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas. Similarly for <=.

_Total 3 instances - 2 file:_

### Instance#1-2:

```solidity
File: contracts/NFTBoostVault.sol
//@audit use userAddresses.length >= 51
343:  if (userAddresses.length > 50) revert NBV_ArrayTooManyElements();

//@audit use change >=1
589:if (change > 0) {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L343C8-L343C74

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L589

### Instance#3:

```solidity
File: contracts/nft/ReputationBadge.sol
//@audit use _claimData.length >= 51
142:  if (_claimData.length > 50) revert RB_ArrayTooLarge();
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L142

## [G-10] <array>.length should not be looked up in every loop of a for-loop
Caching the array length outside a loop saves reading it on each iteration..

_Total 1 instances - 1 file:_

### Instance#1:

```solidity
File: contracts/nft/ReputationBadge.sol
144: for (uint256 i = 0; i < _claimData.length; i++) {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144