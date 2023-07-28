# Gas Optimizations Report

## [G-01] x += y / x -= y costs more gas than x = x + y / x = x - y for state variables

Using the addition operator instead of plus-equals saves **113 gas**. (Same for negation operator instead of minus-equals)
**Gas saved: 1921 gas (per all calls)** 

There are 17 instances of this issue:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L141
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L166
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L171
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L200
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L215
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L242
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L244

```solidity
File: contracts/ARCDVestingVault.sol
141: unassigned.data -= amount;
166: grant.withdrawn += uint128(withdrawable);
171: grant.withdrawn += uint128(remaining);
200: unassigned.data += amount;
215: unassigned.data -= amount;
242: grant.withdrawn += uint128(withdrawable);
244: grant.withdrawn += uint128(amount);
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L117
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L198

```solidity
File: contracts/ArcadeTreasury.sol
117: gscAllowance[token] -= amount;
198: gscAllowance[token] -= amount;
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L161
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L164
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L239
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L241
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L278
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L281
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L505

```solidity
File: contracts/NFTBoostVault.sol
161: balance.data += amount;
164: registration.amount += amount;
239: balance.data -= amount;
241: registration.withdrawn += amount;
278: balance.data += amount;
281: registration.amount += amount;
505: balance.data += _amount;
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L116

```solidity
File: contracts/nft/ReputationBadge.sol
116: amountClaimed[recipient][tokenId] += amount;
```

## [G-02] Remove unnecessary local variable to save gas

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L128

**Gas saved: 5 gas (per call)**

*newVotingPower* variable is assigned value of *amount* variable on Line 128 but *amount* is not updated anywhere after this. Thus we can directly use the *amount* variable on Line 137, 146 and 148 to save gas instead of creating an unnecessary variable.
```solidity
File: contracts/ARCDVestingVault.sol
128: uint128 newVotingPower = amount;
137: grant.latestVotingPower = newVotingPower;
146: votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);
148: emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
```

## [G-03] Public variable only accessed within contract should be marked private 

**Deployment gas cost saved: 120178 gas**

There are 9 instances of this:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/BadgeDescriptor.sol#L24
Deployment gas saved: 35587 gas
```solidity
File: contracts/nft/BadgeDescriptor.sol
24: string public baseURI;
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L34
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L39
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L44
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L49
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L54
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L59
Each instance saves approximately 10500 gas.
Deployment gas saved: 63000 gas  
```solidity
File: contracts/token/ArcadeTokenDistributor.sol
34: bool public treasurySent;
39: bool public devPartnerSent;
44: bool public communityRewardsSent;
49: bool public communityAirdropSent;
54: bool public vestingTeamSent;
59: bool public vestingPartnerSent;
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L94
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L97
Deployment gas saved: 21591 gas
```solidity
File: contracts/token/ArcadeToken.sol
94: address public minter;
97: uint256 public mintingAllowedAfter;
```

## [G-04] Redundant check can be removed to save gas

Total Gas saved: 12 gas (per all calls) **(Opcodes - EQ: 3 gas and OR: 3 gas)**
There are 2 instances of this issue:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L145

The if check below is present in the [airdropReceive() function](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L139) in NFTBoostVault.sol contract. This function is only callable from the airdrop contract (i.e. [the ArcadeMerkleRewards.sol contract](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/libraries/ArcadeMerkleRewards.sol#L88) since ArcadeAirdrop.sol inherits from it). The user parameter passed in through the airdrop contract is the msg.sender, which can never be address(0) as no call can originate from the zero address. Thus, the check can be removed.
```solidity
File: contracts/NFTBoostVault.sol
145: if (user == address(0)) revert NBV_ZeroAddress("user");
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L422

The if check below is present in the [getMultiplier() function](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L418C14-L418C27), which is only called by the [_registerAndDelegate() function](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L462) and [_currentVotingPower() function](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L627). Before calling the getMultiplier() function, both these functions implement a check to ensure the token address and token ID is not zero. Here are the checks for each function:  [_registerAndDelegate()](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L472) and [_currentVotingPower()](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L632). Thus, this check below can be removed as we never enter the if block.
```solidity
File: contracts/NFTBoostVault.sol
422:     if (tokenAddress == address(0) || tokenId == 0) {
423:             return 1e3;
424:     }
```

### Total number of issues: 29
### A] Total deployment cost: 120178 gas saved
### B] Total function execution cost: 1938 gas saved (per all calls)
### Total gas saved [A + B]: 122116 gas saved