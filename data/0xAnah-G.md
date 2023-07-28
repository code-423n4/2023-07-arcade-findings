# Gas Optimizations

## [G-01] Unused Imports
Some files were imported and were not used these files costs gas during deployment and generally this is bad coding practice

## 1 Instance
In the contract below open-zeppelin string library was imported with the statement `import "@openzeppelin/contracts/utils/Strings.sol";` but was not used consider removing this import as it increase deployment gas, pollute the contract's namespace and generally it is bad coding practice. 
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L9
```
Estimated gas saved:
```


## [G-02] Avoid using state or storage variable in emit
In scenarios where you have a memory, calldata variable or parameter with the same value as the state variable you should use the memory, calldata variable or parameter in the emit statement rather than state variable.

### 2 Instances
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136
```solidity
    function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

        minter = _newMinter;
        emit MinterUpdated(minter); //@audit you should emit _newMinter
    }
```
In the `setMinter()` function above rather than emitting the `minter` state variable which would cost `2100` (cold access) we could instead emit the `_newMinter` argument instead. The code could be refactored as shown in the diff below:
```diff
    function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

        minter = _newMinter;
-       emit MinterUpdated(minter);
+       emit MinterUpdated(_newMinter) 
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L148
```solidity
91:     function addGrantAndDelegate(
92:         address who,
93:         uint128 amount,
94:         uint128 cliffAmount,
95:         uint128 startTime,
96:         uint128 expiration,
97:         uint128 cliff,
98:         address delegatee
99:     ) external onlyManager {
100:        // input validation
101:        if (who == address(0)) revert AVV_ZeroAddress("who");
102:        if (amount == 0) revert AVV_InvalidAmount();
103:        // set the new grant
.
.
.
.
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
150:        unassigned.data -= amount;
151:
152:        // update the delegatee's voting power
153:        History.HistoricalBalances memory votingPower = _votingPower();
154:        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
155:        votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);
156:
157:        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
158:    }
```
In the `addGrantAndDelegate()` function above rather than emitting the `grant.delegate` storage variable which would cost `100` gas unit we could instead emit the `delegatee` argument instead which would only cost `3` gas. The code could be refactored as shown in the diff below:
```diff
    function addGrantAndDelegate(
        address who,
        uint128 amount,
        uint128 cliffAmount,
        uint128 startTime,
        uint128 expiration,
        uint128 cliff,
        address delegatee
    ) external onlyManager {
        // input validation
        if (who == address(0)) revert AVV_ZeroAddress("who");
        if (amount == 0) revert AVV_InvalidAmount();
        // set the new grant
                .
                .
                .
                .
        grant.allocation = amount;
        grant.cliffAmount = cliffAmount;
        grant.withdrawn = 0;
        grant.created = startTime;
        grant.expiration = expiration;
        grant.cliff = cliff;
        grant.latestVotingPower = newVotingPower;
        grant.delegatee = delegatee;

        // update the amount of unassigned tokens
        unassigned.data -= amount;

        // update the delegatee's voting power
        History.HistoricalBalances memory votingPower = _votingPower();
        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
-        votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);
+        votingPower.push(delegatee, delegateeVotes + newVotingPower);
        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
    }
```
```
Estimated gas saved: 2100 + 100 = 2200
```


## [G-03] Can transfer 0
In Solidity, performing unnecessary operations can consume more gas than needed, leading to cost inefficiencies. For instance, if a transfer function doesn't have a zero amount check and someone calls it with a zero amount, unnecessary gas will be consumed in executing the function, even though the state of the contract remains the same. By implementing a zero amount check, such unnecessary function calls can be avoided, thereby saving gas and making the contract more efficient.

### 1 Instances
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L201
```solidity
    function deposit(uint256 amount) external onlyManager {
        Storage.Uint256 storage unassigned = _unassigned();
        // update unassigned value
        unassigned.data += amount;
        token.transferFrom(msg.sender, address(this), amount);  //@audit can transfer 0
    }
```
In the function above you should factor in a check for zero amount before the `token.transferFrom(msg.sender, address(this), amount);` statement to eliminate the possibility of performing unnecessary transfer operation that would not change state. The function should be refactored as shown in the diff below:
```diff
    function deposit(uint256 amount) external onlyManager {
+       if (amount == 0)
+           revert zeroAmount();
        Storage.Uint256 storage unassigned = _unassigned();
        // update unassigned value
        unassigned.data += amount;
        token.transferFrom(msg.sender, address(this), amount);
    }
```

## [G-04]  Multiplication by two should use bit shifting
x * 2 is the same as x << 1 but a little cheaper.
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L154
```solidity
154:        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
```
In the code above the value of the `MINT_CAP` constant variable is 2 so we save `4` units of gas if the multiplication is refactored to using bit shifting. The refactored is shown below:
```diff
-        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
+        uint256 mintCapAmount = (totalSupply() << (MINT_CAP - 1)) / PERCENT_DENOMINATOR;
```
```
Estimated gas saved: 4
```


## [G-05] Move lesser gas costing revert checks to the top
`Revert()` statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### 7 Instances
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146-#L148
```solidity
145:    function mint(address _to, uint256 _amount) external onlyMinter {
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
In the `mint()` function above the if statements on line `147` and `148` should be moved to the top of the function since their operations cost lesser gas than the `if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp)` statement which reads from state (2100 gas units) so that in scenarios where any of `if (_to == address(0))` or `if (_amount == 0)` conditionals results to `true` the function could revert before performing the gas consuming opertions of `if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp)` . The function should be refactored as shown in the diff below:
```diff
    function mint(address _to, uint256 _amount) external onlyMinter {
-       if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
-       if (_to == address(0)) revert AT_ZeroAddress("to");
-       if (_amount == 0) revert AT_ZeroMintAmount();

+       if (_amount == 0) revert AT_ZeroMintAmount();
+       if (_to == address(0)) revert AT_ZeroAddress("to");
+       if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
        // record the mint
        mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;

        // inflation cap enforcement - 2% of total supply
        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
        if (_amount > mintCapAmount) {
            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
        }

        _mint(_to, _amount);
    }
``` 

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L74-#L75
```solidity
    function toTreasury(address _treasury) external onlyOwner {
        if (treasurySent) revert AT_AlreadySent();
        if (_treasury == address(0)) revert AT_ZeroAddress("treasury");

        treasurySent = true;

        arcadeToken.safeTransfer(_treasury, treasuryAmount);

        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);
    }
```
In the `toTreasury()` function above the `if (_treasury == address(0)) revert AT_ZeroAddress("treasury")` statement should be moved to the top of the function since it checks if the `_treasury` argument is equal to the zero address and it costs lesser gas than the `if (treasurySent) revert AT_AlreadySent()` statement which reads from state, so that in scenarios where the `_treasury` argument is equal to the zero address the function can revert without executing the gas consuming operation of `if (treasurySent) revert AT_AlreadySent()` thereby saving `2100` units of gas. The code should be refactored as shown in the diff below:
```diff
    function toTreasury(address _treasury) external onlyOwner {
-       if (treasurySent) revert AT_AlreadySent();
-       if (_treasury == address(0)) revert AT_ZeroAddress("treasury");

+       if (_treasury == address(0)) revert AT_ZeroAddress("treasury");
+       if (treasurySent) revert AT_AlreadySent();

        treasurySent = true;

        arcadeToken.safeTransfer(_treasury, treasuryAmount);

        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L90-#L91
```solidity
    function toDevPartner(address _devPartner) external onlyOwner {
        if (devPartnerSent) revert AT_AlreadySent();
        if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");

        devPartnerSent = true;

        arcadeToken.safeTransfer(_devPartner, devPartnerAmount);

        emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);
    }
```
In the `toDevPartner()` function above the `if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner")` statement should be moved to the top of the function since it checks if the `_devPartner` argument is equal to the zero address and it costs lesser gas than the `if (devPartnerSent) revert AT_AlreadySent()` statement which reads from state, so that in scenarios where the `_devPartner` argument is equal to the zero address the function can revert without executing the gas consuming operation of `if (devPartnerSent) revert AT_AlreadySent()` thereby saving `2100` units of gas. The code should be refactored as shown in the diff below:
```diff
    function toDevPartner(address _devPartner) external onlyOwner {
-        if (devPartnerSent) revert AT_AlreadySent();
-        if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");

+        if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");
+        if (devPartnerSent) revert AT_AlreadySent();

         devPartnerSent = true;

         arcadeToken.safeTransfer(_devPartner, devPartnerAmount);

         emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L106-#L107
```solidity
    function toCommunityRewards(address _communityRewards) external onlyOwner {
        if (communityRewardsSent) revert AT_AlreadySent();
        if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");

        communityRewardsSent = true;

        arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);

        emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);
    }
```
In the `toCommunityRewards()` function above the `if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards")` statement should be moved to the top of the function since it checks if the `_communityRewards` argument is equal to the zero address and it costs lesser gas than the `if (communityRewardsSent) revert AT_AlreadySent()` statement which reads from state, so that in scenarios where the `_communityRewards` argument is equal to the zero address the function can revert without executing the gas consuming operation of `if (communityRewardsSent) revert AT_AlreadySent()` thereby saving `2100` units of gas. The code should be refactored as shown in the diff below:
```diff
    function toCommunityRewards(address _communityRewards) external onlyOwner {
-       if (communityRewardsSent) revert AT_AlreadySent();
-       if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");

+       if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");
+       if (communityRewardsSent) revert AT_AlreadySent();

        communityRewardsSent = true;

        arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);

        emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L122-#L123
```solidity
    function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {
        if (communityAirdropSent) revert AT_AlreadySent();
        if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");

        communityAirdropSent = true;

        arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);

        emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);
    }
```
In the `toCommunityAirdrop()` function above the `if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop")` statement should be moved to the top of the function since it checks if the `_communityAirdrop` argument is equal to the zero address and it costs lesser gas than the `if (communityAirdropSent) revert AT_AlreadySent()` statement which reads from state, so that in scenarios where the `_communityAirdrop` argument is equal to the zero address the function can revert without executing the gas consuming operation of `if (communityAirdropSent) revert AT_AlreadySent()` thereby saving `2100` units of gas. The code should be refactored as shown in the diff below:
```diff
    function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {
-       if (communityAirdropSent) revert AT_AlreadySent();
-       if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");

+       if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");
+       if (communityAirdropSent) revert AT_AlreadySent();
        communityAirdropSent = true;

        arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);

        emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L139-#L140
```solidity
    function toTeamVesting(address _vestingTeam) external onlyOwner {
        if (vestingTeamSent) revert AT_AlreadySent();
        if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");

        vestingTeamSent = true;

        arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);

        emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);
    }
```
In the `toTeamVesting()` function above the `if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam")` statement should be moved to the top of the function since it checks if the `_vestingTeam` argument is equal to the zero address and it costs lesser gas than the `if (vestingTeamSent) revert AT_AlreadySent()` statement which reads from state, so that in scenarios where the `_vestingTeam` argument is equal to the zero address the function can revert without executing the gas consuming operation of `if (vestingTeamSent) revert AT_AlreadySent()` thereby saving `2100` units of gas. The code should be refactored as shown in the diff below:
```diff
    function toTeamVesting(address _vestingTeam) external onlyOwner {
-       if (vestingTeamSent) revert AT_AlreadySent();
-       if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");

+       if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");
+       if (vestingTeamSent) revert AT_AlreadySent();

        vestingTeamSent = true;

        arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);

        emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L156-#L157
```solidity
    function toPartnerVesting(address _vestingPartner) external onlyOwner {
        if (vestingPartnerSent) revert AT_AlreadySent();
        if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");

        vestingPartnerSent = true;

        arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);

        emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);
    }
```
In the `toPartnerVesting()` function above the `if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner")` statement should be moved to the top of the function since it checks if the `_vestingPartner` argument is equal to the zero address and it costs lesser gas than the `if (vestingPartnerSent) revert AT_AlreadySent()` statement which reads from state, so that in scenarios where the `_vestingTeam` argument is equal to the zero address the function can revert without executing the gas consuming operation of `if (vestingPartnerSent) revert AT_AlreadySent()` thereby saving `2100` units of gas. The code should be refactored as shown in the diff below:
```diff
    function toPartnerVesting(address _vestingPartner) external onlyOwner {
-       if (vestingPartnerSent) revert AT_AlreadySent();
-       if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");

+       if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");
+       if (vestingPartnerSent) revert AT_AlreadySent();

        vestingPartnerSent = true;

        arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);

        emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);
    }
```
```
Estimated gas saved in the worst case: 2100 * 7 = 14700
```


## [G-06] Declaring redundant variables
Some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size.

### 2 Instances
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L98-#L101
```solidity
    function queryVotePower(address user, uint256 blockNumber, bytes calldata) external override returns (uint256) {
        // Get our reference to historical data
        History.HistoricalBalances memory votingPower = _votingPower();

        // Find the historical data and clear everything more than 'staleBlockLag' into the past
        return votingPower.findAndClear(user, blockNumber, block.number - staleBlockLag);
    }
```
In the `queryVotePower()` function above the `votingPower` variable was declared and used once. `1092` gas units can be saved during deployment and `18` gas units when the function is called by not declaring the `votingPower` variable thus the code could be refactored to:
```solidity
    function queryVotePower(address user, uint256 blockNumber, bytes calldata) external override returns (uint256) {
        
        // Find the historical data and clear everything more than 'staleBlockLag' into the past
        return _votingPower().findAndClear(user, blockNumber, block.number - staleBlockLag);

    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L114-#L117
```solidity
    function queryVotePowerView(address user, uint256 blockNumber) external view returns (uint256) {
        // Get our reference to historical data
        History.HistoricalBalances memory votingPower = _votingPower();

        // Find the historical datum
        return votingPower.find(user, blockNumber);
    }
```
In the `queryVotePowerView()` function above the `votingPower` variable was declared and used once. `2167` gas units can be saved during deployment and `18` gas units when the function is called by not declaring the `votingPower` variable thus the code could be refactored to:
```solidity
    function queryVotePowerView(address user, uint256 blockNumber) external view returns (uint256) {
        
        // Find the historical datum
        return _votingPower().find(user, blockNumber);

    }
```
```
Estimated gas saved: 1092 + 18 + 2167 + 18: 3295
```


## [G-07] Create immutable or constant variable to avoid redundant computations

### 1 Instances
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L697-#L699
```solidity
697:    function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {
698:        return this.onERC1155Received.selector;
699:    }
```
The purpose of the above function is to return its selector whose value is a constant, this function cost `93660` units of deployment gas and `23371` gas units when the function is called. We can eliminate this function and use either a public constant or immutable variable instead.
1. Using an Immutable variable variable: We can do the calculations for the functions selector in the contract and save the value in a public immutable variable using this approach would cost `20619` gas during deployment and `309` gas when the getter function of the immutable variable is used. This approach would save `70634` units of deployment gas and `23026` gas when the function are executed. The code should be refactored to:
```diff
-    function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {
-        return this.onERC1155Received.selector;
-    }

    //This should be move to the top of the contract
+   bytes4 public immutable ONERC1155RECEIVED = bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"));
```
```
Estimated gas saved: 70634 + 23026 = 93660
```
2. Using a constant variable: Since the selector of a function is a value that never changes we could compute the function's selector externally (i.e not in the contract thereby saving the gas that would have been used for the computation) then assign the resulting value to a public constant variable in the contract. Using this approach would cost `20290` gas during deployment and `315` gas when the getter function of the constant variable is used. This approach would save `73370` units of deployment gas and `23056` gas when the function are executed. The code should be refactored to:
```diff
-    function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {
-        return this.onERC1155Received.selector;
-    }

    //This should be move to the top of the contract
+   //bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))
+   bytes4 public constant ONERC1155RECEIVED =  0xf23a6e61;
```
```
Estimated gas saved: 73370 + 23056 = 96426
```


## [G-08]  State variables should be cached in stack variables rather than re-reading them from storage
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.
Please note that these instances were not included in the bots report.

### 7 Instances

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L73-#L82
```solidity
73:    function toTreasury(address _treasury) external onlyOwner {
74:        if (treasurySent) revert AT_AlreadySent();
75:        if (_treasury == address(0)) revert AT_ZeroAddress("treasury")
76:
77:        treasurySent = true;
78:
79:        arcadeToken.safeTransfer(_treasury, treasuryAmount);    //@audit 1st SLOAD 
80:
81:        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);    //@audit 2nd SLOAD
82:    }
```
In the `toTreasury()` function above we could save `100` gas units by caching the state variable `arcadeToken` in a stack variable. The code should be refactored as shown by the diff below:
```diff
    function toTreasury(address _treasury) external onlyOwner {
        if (treasurySent) revert AT_AlreadySent();
        if (_treasury == address(0)) revert AT_ZeroAddress("treasury");

        treasurySent = true;

+       IArcadeToken stackArcadeToken = arcadeToken ;

-       arcadeToken.safeTransfer(_treasury, treasuryAmount);
+       stackArcadeToken.safeTransfer(_treasury, treasuryAmount);

-       emit Distribute(address(arcadeToken), _treasury, treasuryAmount);
+       emit Distribute(address(stackArcadeToken), _treasury, treasuryAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L89-#L98
```solidity
89:    function toDevPartner(address _devPartner) external onlyOwner {
90:        if (devPartnerSent) revert AT_AlreadySent();
91:        if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");
92:
93:        devPartnerSent = true;
94:
95:        arcadeToken.safeTransfer(_devPartner, devPartnerAmount);    //@audit 1st SLOAD 
96:
97:        emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);   @audit 2nd SLOAD
98:    }
```
In the `toDevPartner()` function above we could save `100` gas units by caching the state variable `arcadeToken` in a stack variable. The code should be refactored as shown by the diff below:
```diff
    function toDevPartner(address _devPartner) external onlyOwner {
        if (devPartnerSent) revert AT_AlreadySent();
        if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");

        devPartnerSent = true;

+       IArcadeToken stackArcadeToken = arcadeToken;

-       arcadeToken.safeTransfer(_devPartner, devPartnerAmount);
+       stackArcadeToken.safeTransfer(_devPartner, devPartnerAmount);

-       emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);
+       emit Distribute(address(stackArcadeToken), _devPartner, devPartnerAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L105-#L114
```solidity
105:    function toCommunityRewards(address _communityRewards) external onlyOwner {
106:        if (communityRewardsSent) revert AT_AlreadySent();
107:        if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");
108:
109:        communityRewardsSent = true;
110:
111:        arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);    //@audit 1st SLOAD
112:
113:        emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);   //@audit 2nd SLOAD
114:    }
```
In the `toCommunityRewards()` function above we could save `100` gas units by caching the state variable `arcadeToken` in a stack variable. The code should be refactored as shown by the diff below:
```diff
    function toCommunityRewards(address _communityRewards) external onlyOwner {
        if (communityRewardsSent) revert AT_AlreadySent();
        if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");

        communityRewardsSent = true;

+       IArcadeToken stackArcadeToken = arcadeToken;

-       arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);
+       stackArcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);

-       emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);
+       emit Distribute(address(stackArcadeToken), _communityRewards, communityRewardsAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L121-#L130
```solidity
121:    function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {
122:        if (communityAirdropSent) revert AT_AlreadySent();
123:        if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");
124:
125:        communityAirdropSent = true;
126:
127:        arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);    //@audit 1st SLOAD
128:
129:        emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);   //@audit 2nd SLOAD
130:    }
```
In the `toCommunityAirdrop()` function above we could save `100` gas units by caching the state variable `arcadeToken` in a stack variable. The code should be refactored as shown by the diff below:
```diff
    function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {
        if (communityAirdropSent) revert AT_AlreadySent();
        if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");

        communityAirdropSent = true;

+       IArcadeToken stackArcadeToken = arcadeToken;

-       arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);
+       stackArcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);

-       emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);
+       emit Distribute(address(stackArcadeToken), _communityAirdrop, communityAirdropAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L138-#L147
```solidity
138:    function toTeamVesting(address _vestingTeam) external onlyOwner {
139:        if (vestingTeamSent) revert AT_AlreadySent();
140:        if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");
141:
142:        vestingTeamSent = true;
143:
144:        arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);    //@audit 1st SLOAD
145:
146:        emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);    //@audit 2nd SLOAD
147:    }
```
In the `toTeamVesting()` function above we could save `100` gas units by caching the state variable `arcadeToken` in a stack variable. The code should be refactored as shown by the diff below:
```diff
    function toTeamVesting(address _vestingTeam) external onlyOwner {
        if (vestingTeamSent) revert AT_AlreadySent();
        if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");

        vestingTeamSent = true;

+       IArcadeToken stackArcadeToken = arcadeToken;

-       arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);
+       stackArcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);

-       emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);
+       emit Distribute(address(stackArcadeToken), _vestingTeam, vestingTeamAmount);
    }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L155-#L164
```solidity
155:    function toPartnerVesting(address _vestingPartner) external onlyOwner {
156:        if (vestingPartnerSent) revert AT_AlreadySent();
157:        if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");
158:
159:        vestingPartnerSent = true;
160:
161:        arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);    //@audit 1st SLOAD
162:
163:        emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);       //@audit 2nd SLOAD
164:    }
```
In the `toPartnerVesting()` function above we could save `100` gas units by caching the state variable `arcadeToken` in a stack variable. The code should be refactored as shown by the diff below:
```diff
    function toPartnerVesting(address _vestingPartner) external onlyOwner {
        if (vestingPartnerSent) revert AT_AlreadySent();
        if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");

        vestingPartnerSent = true;

+       IArcadeToken stackArcadeToken = arcadeToken;

-       arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);
+       stackArcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);

-       emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);
+       emit Distribute(address(stackArcadeToken), _vestingPartner, vestingPartnerAmount);
    }
```
```
Estimated gas saved = 6 * 100 = 600 gas units
```


## [G-09] Unchecked Divisions
Divisions which do not divide by -X cannot overflow or underflow so such operations can be unchecked to save gas.

### Instances
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L494
```solidity
494:    uint128 newVotingPower = (_amount * multiplier) / MULTIPLIER_DENOMINATOR;
```
`257` units of gas could be saved if the above calculations is done in an unchecked block. The code should be refactored to:
```diff
-       uint128 newVotingPower = (_amount * multiplier) / MULTIPLIER_DENOMINATOR;

+       uint128 newVotingPower;
+       unchecked{
+           newVotingPower = (_amount * multiplier) / MULTIPLIER_DENOMINATOR;
+       }
```

- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L154
```solidity
154:    uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
```
`257` units of gas could be saved if the above calculations is done in an unchecked block. The code should be refactored to:
```diff
-       uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;

+       uint256 mintCapAmount
+       unchecked{
+           mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
+       }
```
```
Estimated gas saved: 514
```


## [G-10] Unbounded Gas Consumption Risk Due to External Call Recipients
In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using addr.call{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

### 1 Instances
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341
```solidity
333:    function batchCalls(
334:        address[] memory targets,
335:        bytes[] calldata calldatas
336:    ) external onlyRole(ADMIN_ROLE) nonReentrant {
337:        if (targets.length != calldatas.length) revert T_ArrayLengthMismatch();
338:        // execute a package of low level calls
339:        for (uint256 i = 0; i < targets.length; ++i) {
400:            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
401:            (bool success, ) = targets[i].call(calldatas[i]);
402:            // revert if a single call fails
403:            if (!success) revert T_CallFailed();
404:        }
405:    }
```
In the `batchCalls()` function low-level call were made without specifing the amount of gas to be used in the low-level call. This is bad practice as a malicious external function could use up all the gas thereby causing the `batchCalls()` function to revert. You should specify the amount of gas to be used in a low-level call as this would prevent such occurences. The code could be refactored to:
```diff
    function batchCalls(
        address[] memory targets,
        bytes[] calldata calldatas
    ) external onlyRole(ADMIN_ROLE) nonReentrant {
        if (targets.length != calldatas.length) revert T_ArrayLengthMismatch();
        // execute a package of low level calls
        for (uint256 i = 0; i < targets.length; ++i) {
            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
-           (bool success, ) = targets[i].call(calldatas[i]);
+           (bool success, ) = targets[i].call{gas: /*specify amount here*/}(calldatas[i]);
            // revert if a single call fails
            if (!success) revert T_CallFailed();
        }
    }
```
```
Estimated gas saved:
```


## [G-11] Provide functions for batch operations
You should provide a function for batch operations rather calling a function multiple times in a loop. 
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol
```soldity
342:    function updateVotingPower(address[] calldata userAddresses) public override {
343:        if (userAddresses.length > 50) revert NBV_ArrayTooManyElements();
344:
345:        for (uint256 i = 0; i < userAddresses.length; ++i) {
346:            NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[userAddresses[i]];
347:            _syncVotingPower(userAddresses[i], registration);
348:        }
349:    }
```
In the `updateVotingPower()` function above rather than calling `_getRegistrations()` and `_syncVotingPower()` functions multiple times in a loop causing the evm to perform multiple `JUMP` statements per loop index we could provide similar functions but for batch operations thereby saving up to `32` gas per loop .
```diff
-    function updateVotingPower(address[] calldata userAddresses) public override {
-        if (userAddresses.length > 50) revert NBV_ArrayTooManyElements();
-
-        for (uint256 i = 0; i < userAddresses.length; ++i) {
-            NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[userAddresses[i]];
-            _syncVotingPower(userAddresses[i], registration);
-        }
-    }

+    function updateVotingPower(address[] calldata userAddresses) public override {
+        if (userAddresses.length > 50) revert NBV_ArrayTooManyElements();
+
+        
+            NFTBoostVaultStorage.Registration[] storage registrations = _getRegistrationsBatch(userAddresses);
+            _syncVotingPowerBatch(userAddresses, registrations);
+        
+    }

+   function _getRegistrationsBatch(address[] calldata userAddresses) internal pure returns
+   (NFTBoostVaultStorage.Registration[] storage registrations) {
+       uint256 addressesLength = userAddresses.length;
+       for (uint256 i; i < addressesLength; ){
+           registrations[i] = NFTBoostVaultStorage.mappingAddressToRegistrationPtr("registrations")[userAddresses[i]];
+            unchecked{
+                ++i;
+            }
+       }
+        
+   }

+    function _syncVotingPowerBatch(address[] calldata userAddresses, NFTBoostVaultStorage.Registration[] storage registrations) internal {
+       uint256 nRegistrations = registrations.length;
+       for (uint256 i; i < nRegistrations;) {
+           History.HistoricalBalances memory votingPower = _votingPower();
+           uint256 delegateeVotes = votingPower.loadTop(registrations[i].delegatee);
+
+           uint256 newVotingPower = _currentVotingPower(registrations[i]);
+           // get the change in voting power. Negative if the voting power is reduced
+           int256 change = int256(newVotingPower) - int256(uint256(registrations[i].latestVotingPower));
+
+           // do nothing if there is no change
+           if (change == 0) return;
+           if (change > 0) {
+               votingPower.push(registrations[i].delegatee, delegateeVotes + uint256(change));
+           } else {
+               // if the change is negative, we multiply by -1 to avoid underflow when casting
+               votingPower.push(registrations[i].delegatee, delegateeVotes - uint256(change * -1));
+           }
+
+           registration.latestVotingPower = uint128(newVotingPower);
+
+           emit VoteChange(userAddresses[i], registration.delegatee, change);
+
+           unchecked{
+               ++i;
+           }
+
+       }
+   }
```
Please note it could be argued that this change would increase bytecode size and deployment cost but in the longrun it would be beneficial
```
Estimated gas saved: 32 gas per loop
``` 