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

**Deployment cost saved: 35587 gas**

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/BadgeDescriptor.sol#L24
```solidity
File: contracts/nft/BadgeDescriptor.sol
24: string public baseURI;
```

### Total number of issues: 19
### A] Total deployment cost: 35587 gas saved
### B] Total function execution cost: 1926 gas saved (per all calls)
### Total gas saved [A + B]: 37513 gas saved