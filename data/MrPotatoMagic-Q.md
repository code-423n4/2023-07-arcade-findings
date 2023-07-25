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

## [N-03] Redundant comments should be removed to improve code understandability

Incorrect comments should be erased or modified to prevent readers from gaining faulty assumptions about the codebase.

There are 2 instances of this issue:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L116
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L197
```solidity
File: contracts/ArcadeTreasury.sol
116: // Will underflow if amount is greater than remaining allowance
197: // Will underflow if amount is greater than remaining allowance
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

There are 2 instances of this issue:

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

Using a two-step change mechanism (i.e. setting the new address to a temporary variable and letting the new address accept the ownership by calling another function which finally sets the address/role) for both these issues can prevent the problems mentioned.

### Non-Critical issues: 4
### Low severity issues: 1
### Total: 38 instances over 4 issues