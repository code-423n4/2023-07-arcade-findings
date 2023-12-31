# QA Report

## [N-01] Missing event emission for critical changes in functions

The functions below do not emit an event after execution is complete. 

There are 16 instances of this issue:

reclaim() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeAirdrop.sol#L62

addNftAndDelegate() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L114

airdropReceive() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L139

withdraw() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L223

addTokens() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L266

updateNft() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L305

updateVotingPower() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L342

_grantVotingPower() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L521

_lockTokens() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L650

_lockNft() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L673

setTimelock() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L68

setManager() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L80

revokeGrant() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L157

deposit() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L197

withdraw() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L211

claim() - https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L228

## [N-02] Immutable variables should be marked as i_variable to differentiate them from other variables

Changing immutable variables to i_variable improves readability of the codebase.

There are 2 instances of this issue:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L33
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L36
```solidity
File: contracts/BaseVotingVault.sol
33: IERC20 public immutable token;
36: uint256 public immutable staleBlockLag;
```

## [N-03] Redundant comments should be removed/modified to improve code understandability

Incorrect comments should be erased or modified to prevent readers from gaining faulty assumptions about the codebase.

There are 3 instances of this issue:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L116
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L197
These comments incorrectly mention that underflow occurs when underflow does not actually occur but only causes revert (since we use compiler version ^0.8.0). This decreases code understandability during audits or when devs are reading the codebase. The "underflow" word should be replaced with "revert" to better suit the intention of the comment.
```solidity
File: contracts/ArcadeTreasury.sol
116: // Will underflow if amount is greater than remaining allowance
197: // Will underflow if amount is greater than remaining allowance
```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/BadgeDescriptor.sol#L42
We use ERC1155 in the codebase and not ERC721. The comment should be updated with "ERC1155 token ID" for better code understandability.
```solidity
File: contracts/nft/BadgeDescriptor.sol
42: * @notice Getter of specific URI for an ERC721 token ID.
```

## [N-04] Consider using delete rather than assigning zero/false to clear values

The delete keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic

There are 16 instances of this issue:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L251
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L252
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L253
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L254
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L499
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L565
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L566C9-L566C34
```solidity
File: contracts/NFTBoostVault.sol
251: registration.amount = 0;
252: registration.latestVotingPower = 0;
253: registration.withdrawn = 0;
254: registration.delegatee = address(0);
499: registration.withdrawn = 0;
565: registration.tokenAddress = address(0);
566: registration.tokenId = 0;
```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L133
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L178
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L179
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L180
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L181
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L182
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L183
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L184
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L185
```solidity
File: contracts/ARCDVestingVault.sol
133: grant.withdrawn = 0;
178: grant.allocation = 0;
179: grant.cliffAmount = 0;
180: grant.withdrawn = 0;
181: grant.created = 0;
182: grant.expiration = 0;
183: grant.cliff = 0;
184: grant.latestVotingPower = 0;
185: grant.delegatee = address(0);
```

## [L-01] Use two-step change mechanism to change critical addresses

If set incorrectly, these critical addresses cannot be changed and may leave certain functions stagnant.

There are 3 instances of this issue:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L174

The arcadeToken (Line 178) once set, cannot be reset by owner due to the check on Line 176. If this is set incorrectly (intentionally or unintentionally), it can lead to the contract state remaining stagnant since all other token distribution functions in the contract depend on the arcadeToken.
```solidity
File: contracts/token/ArcadeTokenDistributor.sol
174:     function setToken(IArcadeToken _arcadeToken) external onlyOwner {
175:         if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");
176:         if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();
177: 
178:         arcadeToken = _arcadeToken;
179:     }
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132

Only the minter can call this function. Thus if the minter (intentionally or unintentionally) sets the new minter incorrectly, it can leave the mint() function in the contract stagnant and disallows the team to use the inflationary system put in place. For example, it prevents adjustments for changes in cases of token demand (scarcity of tokens) and market conditions (deflationary prices).
```solidity
File: contracts/token/ArcadeToken.sol
132:    function setMinter(address _newMinter) external onlyMinter {
133:         if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");
134: 
135:         minter = _newMinter;
136:         emit MinterUpdated(minter);
137:     }
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L68

The timelock contract is responsible for changing the timelock address. If set incorrectly (intentionally or unintentionally), it can lead to the [setManager() function](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L80C52-L80C64) remaining stagnant (i.e. no one will be able to set a manager for the contract anymore). This has a huge impact as the manager role is responsible for [adding](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L91)/[revoking](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L157) grants and [depositing](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L197)/[withdrawing](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L211) tokens to the ARCDVestingVault.sol contract as well as setting the [airdrop contract](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L392) and [multiplier value associated with an ERC1155 contract address](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L363C108-L363C119) in the NFTBoostVault.sol contract. Additionally, the timelock itself is responsible for the [unlocking of withdrawals](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L378C41-L378C53) in the NFTBoostVault.sol contract. This could lead to the user's ERC20 tokens and ERC1155 NFT being locked in the contract forever.
```solidity
File: contracts/BaseVotingVault.sol
68:     function setTimelock(address timelock_) external onlyTimelock {
69:         if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");
70: 
71:         Storage.set(Storage.addressPtr("timelock"), timelock_);
72:     }
```

Using a two-step change mechanism (i.e. setting the new address to a temporary variable and letting the new address accept the ownership by calling another function which finally sets the address/role) for these issues can prevent the problems mentioned.

## [L-02] Missing input validation of calldata can lead to revert during proposal execution

The _getSelector() function is an internal helper function to get the function selector of a calldata string (calldata provided from the [proposal() function](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/CoreVoting.sol#L154)). But this function does not check if the calldata provided has a length >= 4. This can lead to an invalid function selector being returned to the proposal() function. Additionally since the calldata is invalid, it will lead to the proposal failing during execution. 

[Statement in proposal() function calling _getSelector() function with calldata](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/CoreVoting.sol#L154):
```solidity
File: contracts/external/council/CoreVoting.sol
154: bytes4 selector = _getSelector(calldatas[i]);
```

[_getSelector() function without any calldata validation](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/CoreVoting.sol#L365):
```solidity
File: contracts/external/council/CoreVoting.sol
```solidity
File: contracts/external/council/CoreVoting.sol
365:    function _getSelector(bytes memory _calldata)
366:         internal
367:         pure
368:         returns (bytes4 out)
369:     {
370:         assembly {
371:             out := and(
372:                 mload(add(_calldata, 32)),
373:                 0xFFFFFFFFF0000000000000000000000000000000000000000000000000000000
374:             )
375:         }
376:     }
```

[Proposal execution in execute() function that leads to revert on invalid calldata being called](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/CoreVoting.sol#L297):
```solidity
File: contracts/external/council/CoreVoting.sol
297:      for (uint256 i = 0; i < targets.length; i++) {
298:             (bool success, ) = targets[i].call(calldatas[i]);
299:             require(success, "Call failed");
300: 
301:       }
```


Solution: A check should be implemented to not only validate the calldata input to the _getSelector() function and ensure the correct selector is returned to the proposal() function but also to validate the calldata in general to prevent proposal failure during execution. (Note: Check added on Line 370)
```solidity
File: contracts/external/council/CoreVoting.sol
365:    function _getSelector(bytes memory _calldata)
366:         internal
367:         pure
368:         returns (bytes4 out)
369:     {
370:         require(_calldata.length >= 4, "Invalid input length"); //@audit check added here
371:         assembly {
372:             out := and(
373:                 mload(add(_calldata, 32)),
374:                 0xFFFFFFFFF0000000000000000000000000000000000000000000000000000000
375:             )
376:         }
377:     }
```

## [L-03] Missing SET_ALLOWANCE_COOL_DOWN check allows updating the gscAllowance of a token during the 7-day cool down period

**Impact**: Once the gscAllowance for a token is updated, it should enforce the cool down period of 7 days. But the setThreshold() function does not do that, allowing the gscAllowance of any token to be updated. (Note: Although the setThreshold() function is used to set the spendThresholds of a token, it also sets the gscAllowance of a token if the existing value is higher than the small threshold the admin is setting).

**Vulnerability details**:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L281

The if block below is in the [setThreshold() function](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L269). Unlike the [uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L308) check done in [setGSCAllowance() function](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L303), setThreshold() does not implement the check which allows updating gscAllowance for any token even during the cool down period.
```solidity
File: contracts/ArcadeTreasury.sol
281:     if (thresholds.small < gscAllowance[token]) {
282:             gscAllowance[token] = thresholds.small;
283: 
284:             emit GSCAllowanceUpdated(token, thresholds.small);
285:     }
```
**Solution**: Add an [uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L308) check and update [lastAllowanceSet[token] = uint48(block.timestamp);](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L319C9-L319C59) as done in the [setGSCAllowance() function](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L303). Here is the code embedded with the solution to mitigate this issue:
```solidity
File: contracts/ArcadeTreasury.sol
269:     function setThreshold(address token, SpendThreshold memory thresholds) external onlyRole(ADMIN_ROLE) {
270:         // verify that the token is not the zero address
271:         if (token == address(0)) revert T_ZeroAddress("token");
272:         // verify small threshold is not zero
273:         if (thresholds.small == 0) revert T_ZeroAmount();
274: 
275:         // verify thresholds are ascending from small to large
276:         if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small) {
277:             revert T_ThresholdsNotAscending();
278:         }
279:         //@audit check added below
280:         if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {
281:             revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);
282:         }
283: 
284:         // if gscAllowance is greater than new small threshold, set it to the new small threshold
285:         if (thresholds.small < gscAllowance[token]) {
286:             gscAllowance[token] = thresholds.small;
287:             lastAllowanceSet[token] = uint48(block.timestamp); //@audit allowance state updated here
288:             emit GSCAllowanceUpdated(token, thresholds.small);
289:         }
290: 
291:         // Overwrite the spend limits for specified token
292:         spendThresholds[token] = thresholds;
293: 
294:         emit SpendThresholdsUpdated(token, thresholds);
295:     }
```
This solution prevents updating the gscAllowance of a token which might be under the cooldown period and updates lastAllowanceSet[token] to current block.timestamp if the gscAllowance of the token was updated.


### Non-Critical issues: 4
### Low severity issues: 3
### Total: 42 instances over 7 issues