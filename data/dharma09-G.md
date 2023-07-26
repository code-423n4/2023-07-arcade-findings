# GAS OPTIMIZATIONS

All the gas optimizations are determined based opcodes and sample tests.

| Count | Issues | Instances | Gas Saved |
| --- | --- | --- | --- |
| [[G-1]](#) | Emitting storage values instead of the memory one | 2 | 200 |
| [[G-2]](#) | Use nested if and, avoid multiple check combinations | 5 | 75 |
| [[G-3]](#) | State variables should be cached in stack variables rather than re-reading them from storage | 9 | 1800 |
| [[G-4]](#) | If else and require() statements that check input arguments should be at the top of the function | 9 | 1200 |
| [[G-5]](#) | Use storage values in memory to minimize SLOADs | 1 | 200 |
| [[G-6]](#) | The result of a function call should be cached rather than re-calling the function | 1 | 200 |
| [[G-7]](#) | store variable outside of loop  | 1 | 100 |

`Total Gas save 3775`

## [G-01] **Emitting storage values instead of the memory one**

Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

`Gas saves 200`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L148

```solidity
File: /contracts/ARCDVestingVault.sol

91: function addGrantAndDelegate(
92:        address who,
93:        uint128 amount,
94:        uint128 cliffAmount,
95:        uint128 startTime,
96:        uint128 expiration,
97:        uint128 cliff,
98:        address delegatee
99:    ) external onlyManager {
....
130:        // set the new grant
131:        grant.allocation = amount;
132:        grant.cliffAmount = cliffAmount;
133:        grant.withdrawn = 0;
134:        grant.created = startTime;
135:        grant.expiration = expiration;
136:        grant.cliff = cliff;
137:        grant.latestVotingPower = newVotingPower;
138:        grant.delegatee = delegatee;
139:
140:        // update the amount of unassigned tokens
141:        unassigned.data -= amount;
....         //@auidt grant.delegatee
148:        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
149:    }
```

```diff
File: /contracts/ARCDVestingVault.sol

91: function addGrantAndDelegate(
92:        address who,
93:        uint128 amount,
94:        uint128 cliffAmount,
95:        uint128 startTime,
96:        uint128 expiration,
97:        uint128 cliff,
98:        address delegatee
99:    ) external onlyManager {
....
130:        // set the new grant
131:        grant.allocation = amount;
132:        grant.cliffAmount = cliffAmount;
133:        grant.withdrawn = 0;
134:        grant.created = startTime;
135:        grant.expiration = expiration;
136:        grant.cliff = cliff;
137:        grant.latestVotingPower = newVotingPower;
138:        grant.delegatee = delegatee;
139:
140:        // update the amount of unassigned tokens
141:        unassigned.data -= amount;
....
-148:        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
+148:        emit VoteChange(delegatee, who, int256(uint256(newVotingPower)));
149:    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136

```solidity
File: [/contracts/token/ArcadeToken.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136)

132: function setMinter(address _newMinter) external onlyMinter {
133:        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");
134:
135:        minter = _newMinter;
136:        emit MinterUpdated(minter);  //@audit minter
137:    }
```

```diff
File: [/contracts/token/ArcadeToken.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136)

132: function setMinter(address _newMinter) external onlyMinter {
133:        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");
134:
135:        minter = _newMinter;
-136:        emit MinterUpdated(minter);
+136:        emit MinterUpdated(_newMinter);
137:    }
```

## [G-02] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

Saves `15 GAS, Per instance = 75`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L247

```solidity
File: [/contracts/NFTBoostVault.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L247)

247: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
_withdrawNft();
}
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L317

```solidity
File: /contracts/NFTBoostVault.sol

317: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
318:            // withdraw the current ERC1155 from the registration
319:            _withdrawNft();
320:        }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L472C8-L472C60

```solidity
File: /contracts/NFTBoostVault.sol

472: if (_tokenAddress != address(0) && _tokenId != 0) {
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L632

```solidity
File: /contracts/NFTBoostVault.sol

632: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
633:            return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;
634:        }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L659C9-L662C1

```solidity
File: /contracts/NFTBoostVault.sol

659: if (tokenAddress != address(0) && tokenId != 0) {
660:            _lockNft(from, tokenAddress, tokenId, nftAmount);
661:        }
662:    }
```

## [G-03] State variables should be cached in stack variables rather than re-reading them from storage

Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

`Gas saves(9 instances) 1800`

`mintingAllowedAfter` 

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146

```solidity
File:[/contracts/token/ArcadeToken.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146)

145: function mint(address _to, uint256 _amount) external onlyMinter {
146:        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp); //@audit mintingAllowedAfter
147:        if (_to == address(0)) revert AT_ZeroAddress("to");
148:        if (_amount == 0) revert AT_ZeroMintAmount();
149:
150:        // record the mint
151:        mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;
152:
153:        // inflation cap enforcement - 2% of total supply
154:        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
155:        if (_amount > mintCapAmount) {
156:            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
157:        }
158:
159:        _mint(_to, _amount);
160:    }
```

```diff
File:[/contracts/token/ArcadeToken.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146)

145: function mint(address _to, uint256 _amount) external onlyMinter {
-146:        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
+146:        if (block.timestamp < _mintingAllowedAfter) revert AT_MintingNotStarted(_mintingAllowedAfter, block.timestamp); //@audit mintingAllowedAfter
147:        if (_to == address(0)) revert AT_ZeroAddress("to");
148:        if (_amount == 0) revert AT_ZeroMintAmount();
149:
+           mintingAllowedAfter = _mintingAllowedAfter;
150:        // record the mint
151:        mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;
152:
153:        // inflation cap enforcement - 2% of total supply
154:        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
155:        if (_amount > mintCapAmount) {
156:            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
157:        }
158:
159:        _mint(_to, _amount);
160:    }
```

`minter` 

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L167C4-L170C6

```solidity
File: [/contracts/token/ArcadeToken.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146)

167: modifier onlyMinter() {
168:        if (msg.sender != minter) revert AT_MinterNotCaller(minter);
169:        _;
170:    }
```

```diff
File: [/contracts/token/ArcadeToken.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146)

167: modifier onlyMinter() {
-168:        if (msg.sender != minter) revert AT_MinterNotCaller(minter);
+168:        if (msg.sender != _minter) revert AT_MinterNotCaller(_minter);
169:        _;
+           minter = _minter;
170:    }
```

`arcadeToken`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L73C5-L82C6

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol

73: function toTreasury(address _treasury) external onlyOwner {
74:        if (treasurySent) revert AT_AlreadySent();
75:        if (_treasury == address(0)) revert AT_ZeroAddress("treasury");
76:
77:        treasurySent = true;
78:
79:        arcadeToken.safeTransfer(_treasury, treasuryAmount); //@audit arcadeToken
80:
81:        emit Distribute(address(arcadeToken), _treasury, treasuryAmount); //@audit arcadeToken
82:    }
```

```diff
File: /contracts/token/ArcadeTokenDistributor.sol

73: function toTreasury(address _treasury) external onlyOwner {
74:        if (treasurySent) revert AT_AlreadySent();
75:        if (_treasury == address(0)) revert AT_ZeroAddress("treasury");
76:
77:        treasurySent = true;
78:
+          arcadeToken = _arcadeToken;
-79:        arcadeToken.safeTransfer(_treasury, treasuryAmount);
+79:        _arcadeToken.safeTransfer(_treasury, treasuryAmount);
80:
-81:        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);
+81:        emit Distribute(address(_arcadeToken), _treasury, treasuryAmount);
82:    }
```

`arcadeToken`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L89C3-L99C1

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol

89: function toDevPartner(address _devPartner) external onlyOwner {
90:        if (devPartnerSent) revert AT_AlreadySent();
91:        if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");
92:
93:        devPartnerSent = true;
94:
95:        arcadeToken.safeTransfer(_devPartner, devPartnerAmount); //@audit arcadeToken
96:
97:        emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount); //@audit arcadeToken
98:    }
```

`arcadeToken`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L105C4-L114C6

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol

105: function toCommunityRewards(address _communityRewards) external onlyOwner {
106:        if (communityRewardsSent) revert AT_AlreadySent();
107:        if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");
108:
109:        communityRewardsSent = true;
110:
111:        arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount); //@audit arcadeToken
112:
113:        emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount); //@audit arcadeToken
114:    }
```

`arcadeToken`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L121C5-L130C6

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol

121: function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {
122:        if (communityAirdropSent) revert AT_AlreadySent();
123:        if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");
124:
125:        communityAirdropSent = true;
126:
127:        arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount); //@audit arcadeToken
128:
129:        emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount); //@audit arcadeToken
130:    }
```

`arcadeToken`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L138C1-L147C6

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol

138: function toTeamVesting(address _vestingTeam) external onlyOwner {
139:        if (vestingTeamSent) revert AT_AlreadySent();
140:        if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");
141:
142:        vestingTeamSent = true;
143:
144:        arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount); //@audit arcadeToken
145:
146:        emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount); //@audit arcadeToken
147:    }
```

`arcadeToken`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L155C4-L164C6

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol

155: function toPartnerVesting(address _vestingPartner) external onlyOwner {
156:        if (vestingPartnerSent) revert AT_AlreadySent();
157:        if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");
158:
159:        vestingPartnerSent = true;
160:
161:        arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount); //@audit arcadeToken
162:
163:        emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount); //@audit arcadeToken
164:    }
```

### **Use the existing Local variable/global variable when equal to a state variable to avoid reading from state**

 `arcadeToken`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L176

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol

174: function setToken(IArcadeToken _arcadeToken) external onlyOwner {
175:        if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");
176:        if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();
177:
178:        arcadeToken = _arcadeToken;
179:    }
```

```diff
File: /contracts/token/ArcadeTokenDistributor.sol

174: function setToken(IArcadeToken _arcadeToken) external onlyOwner {
175:        if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");
-176:        if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();
+176:        if (address(_arcadeToken) != address(0)) revert AT_TokenAlreadySet();
177:
178:        arcadeToken = _arcadeToken;
179:    }
```

### [G-04] IF’s/require() statements that check input arguments should be at the top of the function

`gas save 1200`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L225

```solidity
File: /contracts/NFTBoostVault.sol

225: if (amount == 0) revert NBV_ZeroAmount();
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L147C7-L148C54

```solidity
File: /contracts/token/ArcadeToken.sol

147: if (_to == address(0)) revert AT_ZeroAddress("to");
148:        if (_amount == 0) revert AT_ZeroMintAmount();
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol

75: if (_treasury == address(0)) revert AT_ZeroAddress("treasury");
91: if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");
107: if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");
123: if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");
140: if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");
157: if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");
```

## [G-05] **Use storage values in memory to minimize SLOADs**

The code can be optimized by minimizing the number of SLOADs.

**`(save 2 SLOAD: 200 gas)`**

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L91

```solidity
File: /contracts/ARCDVestingVault.sol

91: function addGrantAndDelegate(
92:        address who,
93:        uint128 amount,
94:        uint128 cliffAmount,
95:        uint128 startTime,
96:        uint128 expiration,
97:        uint128 cliff,
98:        address delegatee
99:    ) external onlyManager {
....
130:        // set the new grant
131:        grant.allocation = amount;
132:        grant.cliffAmount = cliffAmount;
133:        grant.withdrawn = 0;
134:        grant.created = startTime;
135:        grant.expiration = expiration;
136:        grant.cliff = cliff;
137:        grant.latestVotingPower = newVotingPower;
138:        grant.delegatee = delegatee;
139:
140:        // update the amount of unassigned tokens
141:        unassigned.data -= amount;
142:
143:        // update the delegatee's voting power
144:        History.HistoricalBalances memory votingPower = _votingPower();
145:        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);   //@audit grant.delegatee see line 138
146:        votingPower.push(grant.delegatee, delegateeVotes + newVotingPower); //@audit grant.delegatee
147:
148:        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
149:    }
```

```diff
File: /contracts/ARCDVestingVault.sol

91: function addGrantAndDelegate(
92:        address who,
93:        uint128 amount,
94:        uint128 cliffAmount,
95:        uint128 startTime,
96:        uint128 expiration,
97:        uint128 cliff,
98:        address delegatee
99:    ) external onlyManager {
....
130:        // set the new grant
131:        grant.allocation = amount;
132:        grant.cliffAmount = cliffAmount;
133:        grant.withdrawn = 0;
134:        grant.created = startTime;
135:        grant.expiration = expiration;
136:        grant.cliff = cliff;
137:        grant.latestVotingPower = newVotingPower;
138:        grant.delegatee = delegatee;
139:
140:        // update the amount of unassigned tokens
141:        unassigned.data -= amount;
142:
143:        // update the delegatee's voting power
144:        History.HistoricalBalances memory votingPower = _votingPower();
-145:        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
+145:        uint256 delegateeVotes = votingPower.loadTop(delegatee);
-146:        votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);
+146:        votingPower.push(delegatee, delegateeVotes + newVotingPower);
147:
148:        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
149:    }
```

### [G-07] The result of a function call should be cached rather than re-calling the function

External calls are expensive. Consider caching the following:

### `ArcadeToken.sol.mint()` : **Results of `totalsupply()` should be cached**

`(Save 2 **SLOAD: 200 gas**)`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L145C5-L160C6

```solidity
File: contracts/token/ArcadeToken.sol

145: function mint(address _to, uint256 _amount) external onlyMinter {
146:        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
147:        if (_to == address(0)) revert AT_ZeroAddress("to");
148:        if (_amount == 0) revert AT_ZeroMintAmount();
149:
150:        // record the mint
151:        mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;
152:
153:        // inflation cap enforcement - 2% of total supply
154:        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
155:        if (_amount > mintCapAmount) {
156:            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
157:        }
158:
159:        _mint(_to, _amount);
160:    }
```

```diff
File: contracts/token/ArcadeToken.sol

145: function mint(address _to, uint256 _amount) external onlyMinter {
146:        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
147:        if (_to == address(0)) revert AT_ZeroAddress("to");
148:        if (_amount == 0) revert AT_ZeroMintAmount();
149:
150:        // record the mint
151:        mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;
152:
+           uint256 _totalSupply = totalSupply();
153:        // inflation cap enforcement - 2% of total supply
-154:        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
+154:        uint256 mintCapAmount = (_totalSupply * MINT_CAP) / PERCENT_DENOMINATOR;
155:        if (_amount > mintCapAmount) {
-156:            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
+156:            revert AT_MintingCapExceeded(_totalSupply, mintCapAmount, _amount);
157:        }
158:
159:        _mint(_to, _amount);
160:    }
```

### [G-07] store variable outside the loop

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

`gas save avg 100`

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L342C3-L349C6

```solidity
File: /contracts/NFTBoostVault.sol

342: function updateVotingPower(address[] calldata userAddresses) public override {
343:        if (userAddresses.length > 50) revert NBV_ArrayTooManyElements();
344:
345:        for (uint256 i = 0; i < userAddresses.length; ++i) {
346:            NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[userAddresses[i]]; //@audit 
347:            _syncVotingPower(userAddresses[i], registration);
348:        }
349:    }
```