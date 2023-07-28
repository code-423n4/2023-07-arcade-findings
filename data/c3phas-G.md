# Codebase Optimization Report

## Auditor's Disclaimer

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Table Of Contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Table Of Contents](#table-of-contents)
  - [Note on Gas estimates](#note-on-gas-estimates)
  - [\[G-01\] Tighly pack storage variables](#g-01-tighly-pack-storage-variables)
    - [`mintingAllowedAfter` is a timestamp, we can reduce the size to `uint64` and pack with address `minter` Saves 1 SLOT: 2000 gas](#mintingallowedafter-is-a-timestamp-we-can-reduce-the-size-to-uint64-and-pack-with-address-minter-saves-1-slot-2000-gas)
  - [\[G-02\]When  variables are declared  with the storage keyword ,caching any fields that need to be re-read in stack variables Saves gas](#g-02when--variables-are-declared--with-the-storage-keyword-caching-any-fields-that-need-to-be-re-read-in-stack-variables-saves-gas)
    - [\[G-02-1\]Since we are assigning `grant.delegatee` the value `delegatee`  we should retrieve the function parameter instead of reading the storage variable  - Save 465 Gas](#g-02-1since-we-are-assigning-grantdelegatee-the-value-delegatee--we-should-retrieve-the-function-parameter-instead-of-reading-the-storage-variable----save-465-gas)
    - [\[G-02-2\]Function withdraw can be more optimized by caching the repeatedly accessed data (Save 104 Gas)](#g-02-2function-withdraw-can-be-more-optimized-by-caching-the-repeatedly-accessed-data-save-104-gas)
    - [\[G-02-3\]Optimize function delegate(Saves 568 Gas)](#g-02-3optimize-function-delegatesaves-568-gas)
    - [\[G-02-4\]`grant.delegatee` should be cached(Saves 214 Gas)](#g-02-4grantdelegatee-should-be-cachedsaves-214-gas)
    - [\[G-02-5\]Cache `registration.delegatee` and `registration.latestVotingPower`(Saves 470 Gas)](#g-02-5cache-registrationdelegatee-and-registrationlatestvotingpowersaves-470-gas)
    - [\[G-02-6\]`registration.tokenAddress` and `registration.tokenId ` should be cached (Saves 215 Gas)](#g-02-6registrationtokenaddress-and-registrationtokenid--should-be-cached-saves-215-gas)
    - [\[G-02-7\]`registration.delegatee` should be cached(Save 127 Gas on average)](#g-02-7registrationdelegatee-should-be-cachedsave-127-gas-on-average)
  - [\[G-03\]Cache storage values in memory to minimize SLOADs](#g-03cache-storage-values-in-memory-to-minimize-sloads)
    - [\[G-03-1\]ArcadeTokenDistributor.sol.toTreasury(): Cache `arcadeToken` in memory - Saves 116 Gas on average](#g-03-1arcadetokendistributorsoltotreasury-cache-arcadetoken-in-memory---saves-116-gas-on-average)
    - [\[G-03-2\]Function `toDevPartner` Cache `arcadeToken` saves 105 Gas on average](#g-03-2function-todevpartner-cache-arcadetoken-saves-105-gas-on-average)
    - [\[G-03-3\]Function `toCommunityRewards()` cache `arcadeToken` to save 124 Gas on average](#g-03-3function-tocommunityrewards-cache-arcadetoken-to-save-124-gas-on-average)
    - [\[G-03-4\]Function `toCommunityAirdrop()`: Cache `arcadeToken` to save 116 Gas on average](#g-03-4function-tocommunityairdrop-cache-arcadetoken-to-save-116-gas-on-average)
    - [\[G-03-5\]Function `toTeamVesting()`: cache `arcadeToken` to save 116 Gas on average on average](#g-03-5function-toteamvesting-cache-arcadetoken-to-save-116-gas-on-average-on-average)
    - [\[G-03-6\]Function `toPartnerVesting()` : cache `arcadeToken` to save 116 Gas on average](#g-03-6function-topartnervesting--cache-arcadetoken-to-save-116-gas-on-average)
  - [\[G-04\]We can make the following functions more optimal](#g-04we-can-make-the-following-functions-more-optimal)
    - [Function `getMultiplier()` can be benefit from some refactoring](#function-getmultiplier-can-be-benefit-from-some-refactoring)
  - [\[G-05\]Inline the modifier for optimal checks](#g-05inline-the-modifier-for-optimal-checks)
  - [\[G-06\]Emitting storage values instead of the memory one.](#g-06emitting-storage-values-instead-of-the-memory-one)
    - [Emit `_newMinter` instead of `minter`](#emit-_newminter-instead-of-minter)
  - [\[G-07\]Optimizing check order for cost efficient function execution](#g-07optimizing-check-order-for-cost-efficient-function-execution)
    - [\[G-07-1\]Avoid reading state variables if we can revert on function params](#g-07-1avoid-reading-state-variables-if-we-can-revert-on-function-params)
    - [\[G-07-2\]Validate function parameters before reading any state variable](#g-07-2validate-function-parameters-before-reading-any-state-variable)
    - [\[G-07-3\]Let's validate function parameter `_devPartner` before trying to read the state variable `devPartnerSent`](#g-07-3lets-validate-function-parameter-_devpartner-before-trying-to-read-the-state-variable-devpartnersent)
    - [\[G-07-4\]Checks should be reordered to validate function parameters first](#g-07-4checks-should-be-reordered-to-validate-function-parameters-first)
    - [\[G-07-5\]Reorder the checks to validate the function parameter first](#g-07-5reorder-the-checks-to-validate-the-function-parameter-first)
    - [\[G-07-6\]We should check the validity of `_vestingTeam` before reading from state](#g-07-6we-should-check-the-validity-of-_vestingteam-before-reading-from-state)
    - [\[G-07-7\]Validate the parameter `_vestingPartner` before checking the state variable](#g-07-7validate-the-parameter-_vestingpartner-before-checking-the-state-variable)
    - [\[G-07-8\]Validate on the fly (no need to load all states then validate later - load while validating)](#g-07-8validate-on-the-fly-no-need-to-load-all-states-then-validate-later---load-while-validating)
    - [\[G-07-9\]Validate functional parameter before making the function call](#g-07-9validate-functional-parameter-before-making-the-function-call)
  - [\[G-08\]Don't cache if used once](#g-08dont-cache-if-used-once)
  - [\[G-09\]Result of function call should be cached rather than call the function more than once](#g-09result-of-function-call-should-be-cached-rather-than-call-the-function-more-than-once)
    - [We can cache the results of `totalSupply()` Sad path only](#we-can-cache-the-results-of-totalsupply-sad-path-only)
  - [\[G-10\]Nested if is cheaper than single statement](#g-10nested-if-is-cheaper-than-single-statement)
  - [Conclusion](#conclusion)


## Note on Gas estimates

I've tried to give the exact amount of gas being saved from running the included tests. 
Some functions are not covered by the test cases or are internal/private functions. In this case, the gas can be estimated by looking at the function that utilizes them or by checking the opcodes involved. 

**In some cases , one issue title will have many instances under it, the main issue will be labeled as `G-mainTitleNumber` while the instances follows the format `G-mainTitleNumber-InstanceNumber`**


## [G-01] Tighly pack storage variables

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. 
Here, the storage variables can be tightly packed

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L93-L97

### `mintingAllowedAfter` is a timestamp, we can reduce the size to `uint64` and pack with address `minter` Saves 1 SLOT: 2000 gas

`Uint64` should be big enough to handle timestamps

```solidity
File: /contracts/token/ArcadeToken.sol
93:    /// @notice Minter contract address responsible for minting future tokens
94:    address public minter;

96:    /// @notice The timestamp after which minting may occur
97:    uint256 public mintingAllowedAfter;
```

```diff
diff --git a/contracts/token/ArcadeToken.sol b/contracts/token/ArcadeToken.sol
index 8cbf058..2f35bac 100644
--- a/contracts/token/ArcadeToken.sol
+++ b/contracts/token/ArcadeToken.sol
@@ -94,7 +94,7 @@ contract ArcadeToken is ERC20, ERC20Burnable, IArcadeToken, ERC20Permit {
     address public minter;

     /// @notice The timestamp after which minting may occur
-    uint256 public mintingAllowedAfter;
+    uint64 public mintingAllowedAfter;

```

## [G-02]When  variables are declared  with the storage keyword ,caching any fields that need to be re-read in stack variables Saves gas

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L91-L149

### [G-02-1]Since we are assigning `grant.delegatee` the value `delegatee`  we should retrieve the function parameter instead of reading the storage variable  - Save 465 Gas

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | 186507    |191557   | 186987 | 
| After  | 186042    | 191092   | 186522| 

**We can also cache `unassigned.data` instead of accessing it more than once**
```solidity
File: /contracts/ARCDVestingVault.sol
114:        Storage.Uint256 storage unassigned = _unassigned();
115:        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
    
130:        // set the new grant
131:        grant.allocation = amount;
132:        grant.cliffAmount = cliffAmount;
133:        grant.withdrawn = 0;
134:        grant.created = startTime;
135:        grant.expiration = expiration;
136:        grant.cliff = cliff;
137:        grant.latestVotingPower = newVotingPower;
138:        grant.delegatee = delegatee;

140:        // update the amount of unassigned tokens
141:        unassigned.data -= amount;

145:        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
146:        votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);

148:        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
149:    }
```
We set the storage variable `grant.delegatee` equal to `delegatee` which is a function parameter. Instead of reading the state variable in the later operations, we can just call `delegatee` as it would be cheaper to read. We save 3 sloads
```diff
diff --git a/contracts/ARCDVestingVault.sol b/contracts/ARCDVestingVault.sol
index 48885c6..a9afdae 100644
--- a/contracts/ARCDVestingVault.sol
+++ b/contracts/ARCDVestingVault.sol
@@ -112,7 +112,8 @@ contract ARCDVestingVault is IARCDVestingVault, HashedStorageReentrancyBlock, Ba
         if (cliffAmount >= amount) revert AVV_InvalidCliffAmount();

         Storage.Uint256 storage unassigned = _unassigned();
-        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
+        uint256 unassignedData = unassigned.data;
+        if (unassignedData < amount) revert AVV_InsufficientBalance(unassignedData);

         // load the grant
         ARCDVestingVaultStorage.Grant storage grant = _grants()[who];
@@ -138,14 +139,14 @@ contract ARCDVestingVault is IARCDVestingVault, HashedStorageReentrancyBlock, Ba
         grant.delegatee = delegatee;

         // update the amount of unassigned tokens
-        unassigned.data -= amount;
+        unassigned.data = unassignedData - amount;

         // update the delegatee's voting power
         History.HistoricalBalances memory votingPower = _votingPower();
-        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
-        votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);
+        uint256 delegateeVotes = votingPower.loadTop(delegatee);
+        votingPower.push(delegatee, delegateeVotes + newVotingPower);

-        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
+        emit VoteChange(delegatee, who, int256(uint256(newVotingPower)));
     }

```


https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L211-L218

### [G-02-2]Function withdraw can be more optimized by caching the repeatedly accessed data (Save 104 Gas)

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | -    | -   | 37724 | 
| After  | -    | -   | 37620| 

```solidity
File: /contracts/ARCDVestingVault.sol
211:    function withdraw(uint256 amount, address recipient) external override onlyManager {
212:        Storage.Uint256 storage unassigned = _unassigned();
213:        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
214:        // update unassigned value
215:        unassigned.data -= amount;

217:        token.safeTransfer(recipient, amount);
218:    }
```

```diff
diff --git a/contracts/ARCDVestingVault.sol b/contracts/ARCDVestingVault.sol
index 48885c6..e710ca2 100644
--- a/contracts/ARCDVestingVault.sol
+++ b/contracts/ARCDVestingVault.sol
@@ -210,9 +210,10 @@ contract ARCDVestingVault is IARCDVestingVault, HashedStorageReentrancyBlock, Ba
      */
     function withdraw(uint256 amount, address recipient) external override onlyManager {
         Storage.Uint256 storage unassigned = _unassigned();
-        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
+        uint256 _unassignedData = unassigned.data;
+        if (_unassignedData < amount) revert AVV_InsufficientBalance(_unassignedData);
         // update unassigned value
-        unassigned.data -= amount;
+        unassigned.data = _unassignedData - amount;

         token.safeTransfer(recipient, amount);
     }
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L260-L282

### [G-02-3]Optimize function delegate(Saves 568 Gas)

**We can cache the result of `grant.delegatee` and `grant.latestVotingPower`

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | -    | -   | 110426 | 
| After  | -    | -   | 109858 | 

```solidity
File: /contracts/ARCDVestingVault.sol
260:    function delegate(address to) external {
262:        ARCDVestingVaultStorage.Grant storage grant = _grants()[msg.sender];
262:        if (to == grant.delegatee) revert AVV_AlreadyDelegated();

264:        History.HistoricalBalances memory votingPower = _votingPower();
265:        uint256 oldDelegateeVotes = votingPower.loadTop(grant.delegatee);

268:      votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);
269:        emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));

273:        uint256 newDelegateeVotes = votingPower.loadTop(to);

276:        votingPower.push(to, newDelegateeVotes + grant.latestVotingPower);

279:        grant.delegatee = to;

281:        emit VoteChange(to, msg.sender, int256(uint256(grant.latestVotingPower)));
282:    }
```

```diff
diff --git a/contracts/ARCDVestingVault.sol b/contracts/ARCDVestingVault.sol
index 48885c6..ca1b595 100644
--- a/contracts/ARCDVestingVault.sol
+++ b/contracts/ARCDVestingVault.sol
@@ -261,24 +261,27 @@ contract ARCDVestingVault is IARCDVestingVault, HashedStorageReentrancyBlock, Ba
         ARCDVestingVaultStorage.Grant storage grant = _grants()[msg.sender];
         if (to == grant.delegatee) revert AVV_AlreadyDelegated();

+        address _grantDelegatee = grant.delegatee;
+        uint256 _grantLatestVotingPower = grant.latestVotingPower;
+
         History.HistoricalBalances memory votingPower = _votingPower();
-        uint256 oldDelegateeVotes = votingPower.loadTop(grant.delegatee);
+        uint256 oldDelegateeVotes = votingPower.loadTop(_grantDelegatee);

         // Remove old delegatee's voting power and emit event
-        votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);
-        emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));
+        votingPower.push(_grantDelegatee, oldDelegateeVotes - _grantLatestVotin
gPower);
+        emit VoteChange(_grantDelegatee, msg.sender, -1 * int256(_grantLatestVotingPower));

         // Note - It is important that this is loaded here and not before the previous state change because if
         // to == grant.delegatee and re-delegation was allowed we could be working with out of date state.
         uint256 newDelegateeVotes = votingPower.loadTop(to);

         // add voting power to the target delegatee and emit event
-        votingPower.push(to, newDelegateeVotes + grant.latestVotingPower);
+        votingPower.push(to, newDelegateeVotes + _grantLatestVotingPower);

         // update grant delgatee info
         grant.delegatee = to;

-        emit VoteChange(to, msg.sender, int256(uint256(grant.latestVotingPower)));
+        emit VoteChange(to, msg.sender, int256(uint256(_grantLatestVotingPower)));
     }

```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L341-L357

### [G-02-4]`grant.delegatee` should be cached(Saves 214 Gas)

As this is internal , the gas benchmarks are based on the function `claim()` which calls this function

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | 83862    | 111166   | 98971 | 
| After  | 83648    | 110952   | 98757 | 

```solidity
File: /contracts/ARCDVestingVault.sol
341:    function _syncVotingPower(address who, ARCDVestingVaultStorage.Grant storage grant) internal {
342:        History.HistoricalBalances memory votingPower = _votingPower();

344:        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);

346:        uint256 newVotingPower = grant.allocation - grant.withdrawn;

350:        int256 change = int256(newVotingPower) - int256(grant.latestVotingPower);
351:        // we multiply by -1 to avoid underflow when casting
352:        votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));

354:        grant.latestVotingPower = newVotingPower;

356:        emit VoteChange(grant.delegatee, who, change);
357:    }
```

```diff
diff --git a/contracts/ARCDVestingVault.sol b/contracts/ARCDVestingVault.sol
index 48885c6..9102a2e 100644
--- a/contracts/ARCDVestingVault.sol
+++ b/contracts/ARCDVestingVault.sol
@@ -341,7 +341,9 @@ contract ARCDVestingVault is IARCDVestingVault, HashedStorageReentrancyBlock, Ba
     function _syncVotingPower(address who, ARCDVestingVaultStorage.Grant storage grant) internal {
         History.HistoricalBalances memory votingPower = _votingPower();

-        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
+        address _grantDelegatee = grant.delegatee;
+
+        uint256 delegateeVotes = votingPower.loadTop(_grantDelegatee);

         uint256 newVotingPower = grant.allocation - grant.withdrawn;

@@ -349,11 +351,11 @@ contract ARCDVestingVault is IARCDVestingVault, HashedStorageReentrancyBlock, Ba
         // since the sync is only called when tokens are claimed or grant revoked
         int256 change = int256(newVotingPower) - int256(grant.latestVotingPower
);
         // we multiply by -1 to avoid underflow when casting
-        votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));
+        votingPower.push(_grantDelegatee delegateeVotes - uint256(change * -1));

         grant.latestVotingPower = newVotingPower;

-        emit VoteChange(grant.delegatee, who, change);
+        emit VoteChange(_grantDelegatee, who, change);
     }

     /**
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L182-L212

### [G-02-5]Cache `registration.delegatee` and `registration.latestVotingPower`(Saves 470 Gas)

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | -    | -   | 118494 | 
| After  | -    | -   | 118024 | 

```solidity
File: /contracts/NFTBoostVault.sol
182:    function delegate(address to) external override {
183:        if (to == address(0)) revert NBV_ZeroAddress("to");

185:        NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

188:        if (to == registration.delegatee) revert NBV_AlreadyDelegated();

190:        History.HistoricalBalances memory votingPower = _votingPower();
191:        uint256 oldDelegateeVotes = votingPower.loadTop(registration.delegatee);

194:        votingPower.push(registration.delegatee, oldDelegateeVotes - registration.latestVotingPower);
195:        emit VoteChange(msg.sender, registration.delegatee, -1 * int256(uint256(registration.latestVotingPower)));

199:        uint256 newDelegateeVotes = votingPower.loadTop(to);

202:        uint256 addedVotingPower = _currentVotingPower(registration);

205:        votingPower.push(to, newDelegateeVotes + addedVotingPower);

208:        registration.latestVotingPower = uint128(addedVotingPower);
209:        registration.delegatee = to;

211:        emit VoteChange(msg.sender, to, int256(addedVotingPower));
212:    }
```

```diff
diff --git a/contracts/NFTBoostVault.sol b/contracts/NFTBoostVault.sol
index 5f907ee..bfd8d33 100644
--- a/contracts/NFTBoostVault.sol
+++ b/contracts/NFTBoostVault.sol
@@ -188,11 +188,13 @@ contract NFTBoostVault is INFTBoostVault, BaseVotingVault {
         if (to == registration.delegatee) revert NBV_AlreadyDelegated();

         History.HistoricalBalances memory votingPower = _votingPower();
-        uint256 oldDelegateeVotes = votingPower.loadTop(registration.delegatee);
+        address _registrationDelegatee = registration.delegatee;
+        uint128 _registrationLatestVotingPower = registration.latestVotingPower;
+        uint256 oldDelegateeVotes = votingPower.loadTop(_registrationDelegatee);

         // Remove voting power from old delegatee and emit event
-        votingPower.push(registration.delegatee, oldDelegateeVotes - registration.latestVotingPower);
-        emit VoteChange(msg.sender, registration.delegatee, -1 * int256(uint256(registration.latestVotingPower)));
+        votingPower.push(_registrationDelegatee, oldDelegateeVotes - _registrationLatestVotingPower);
+        emit VoteChange(msg.sender, _registrationDelegatee, -1 * int256(uint256(_registrationLatestVotingPower)));

         // Note - It is important that this is loaded here and not before the previous state change because if
         // to == registration.delegatee and re-delegation was allowed we could
be working with out of date state
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L548-L570

### [G-02-6]`registration.tokenAddress` and `registration.tokenId ` should be cached (Saves 215 Gas)

**The gas benchmarks are based on external function `withdrawNft()` which calls our internal function

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | 97583    | 102383   | 100783 | 
| After  | 97367    | 102167   | 100567 | 

```solidity
File: /contracts/NFTBoostVault.sol
548:    function _withdrawNft() internal {
549:        // load the registration
550:        NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

552:        if (registration.tokenAddress == address(0) || registration.tokenId == 0)
553:            revert NBV_InvalidNft(registration.tokenAddress, registration.tokenId);

556:        IERC1155(registration.tokenAddress).safeTransferFrom(
557:            address(this),
558:            msg.sender,
559:            registration.tokenId,
560:            1,
561:            bytes("")
562:        );
```

```diff
diff --git a/contracts/NFTBoostVault.sol b/contracts/NFTBoostVault.sol
index 5f907ee..934fd10 100644
--- a/contracts/NFTBoostVault.sol
+++ b/contracts/NFTBoostVault.sol
@@ -549,14 +549,17 @@ contract NFTBoostVault is INFTBoostVault, BaseVotingVault {
         // load the registration
         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

-        if (registration.tokenAddress == address(0) || registration.tokenId == 0)
-            revert NBV_InvalidNft(registration.tokenAddress, registration.tokenId);
+        address _registrationTokenAddress = registration.tokenAddress;
+        uint128 _registrationTokenId = registration.tokenId;
+
+        if (_registrationTokenAddress == address(0) || _registrationTokenId == 0)
+            revert NBV_InvalidNft(_registrationTokenAddresss, _registrationTokenId);

         // transfer ERC1155 back to the user
-        IERC1155(registration.tokenAddress).safeTransferFrom(
+        IERC1155(_registrationTokenAddress).safeTransferFrom(
             address(this),
             msg.sender,
-            registration.tokenId,
+            _registrationTokenId,
             1,
             bytes("")
         );
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L579-L599

### [G-02-7]`registration.delegatee` should be cached(Save 127 Gas on average)

**Gas benchmarks based on function `withdraw()` which calls our function**

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | 99036    | 129642   | 112883 | 
| After  | 98856    | 129629   | 112756 | 

```solidity
File: /contracts/NFTBoostVault.sol
579:    function _syncVotingPower(address who, NFTBoostVaultStorage.Registration storage registration) internal {
580:        History.HistoricalBalances memory votingPower = _votingPower();
581:        uint256 delegateeVotes = votingPower.loadTop(registration.delegatee);

583:        uint256 newVotingPower = _currentVotingPower(registration);

585:        int256 change = int256(newVotingPower) - int256(uint256(registration.latestVotingPower));

588:        if (change == 0) return;
589:        if (change > 0) {
590:            votingPower.push(registration.delegatee, delegateeVotes + uint256(change));
591:        } else {

593:            votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));
594:        }

596:        registration.latestVotingPower = uint128(newVotingPower);

598:        emit VoteChange(who, registration.delegatee, change);
599:    }
```

```diff
diff --git a/contracts/NFTBoostVault.sol b/contracts/NFTBoostVault.sol
index 5f907ee..56bb1bc 100644
--- a/contracts/NFTBoostVault.sol
+++ b/contracts/NFTBoostVault.sol
@@ -578,7 +578,8 @@ contract NFTBoostVault is INFTBoostVault, BaseVotingVault {
      */
     function _syncVotingPower(address who, NFTBoostVaultStorage.Registration storage registration) internal {
         History.HistoricalBalances memory votingPower = _votingPower();
-        uint256 delegateeVotes = votingPower.loadTop(registration.delegatee);
+        address _registrationDelegatee = registration.delegatee;
+        uint256 delegateeVotes = votingPower.loadTop(_registrationDelegatee);

         uint256 newVotingPower = _currentVotingPower(registration);
         // get the change in voting power. Negative if the voting power is reduced
@@ -587,15 +588,15 @@ contract NFTBoostVault is INFTBoostVault, BaseVotingVault {
         // do nothing if there is no change
         if (change == 0) return;
         if (change > 0) {
-            votingPower.push(registration.delegatee, delegateeVotes + uint256(c
hange));
+            votingPower.push(_registrationDelegatee, delegateeVotes + uint256(change));
         } else {
             // if the change is negative, we multiply by -1 to avoid underflow when casting
-            votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));
+            votingPower.push(_registrationDelegatee, delegateeVotes - uint256(change * -1));
         }

         registration.latestVotingPower = uint128(newVotingPower);

-        emit VoteChange(who, registration.delegatee, change);
+        emit VoteChange(who, _registrationDelegatee, change);
     }

```

## [G-03]Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L73-L82

### [G-03-1]ArcadeTokenDistributor.sol.toTreasury(): Cache `arcadeToken` in memory - Saves 116 Gas on average

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | 64654    | 64666   | 64665 | 
| After  | 64538    | 64550   | 64549 | 

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
73:    function toTreasury(address _treasury) external onlyOwner {

79:        arcadeToken.safeTransfer(_treasury, treasuryAmount);

81:        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);
82:    }
```

```diff
--- a/contracts/token/ArcadeTokenDistributor.sol
+++ b/contracts/token/ArcadeTokenDistributor.sol
@@ -75,10 +75,11 @@ contract ArcadeTokenDistributor is Ownable {
         if (_treasury == address(0)) revert AT_ZeroAddress("treasury");

         treasurySent = true;
+        IArcadeToken  _arcadeToken = arcadeToken;

-        arcadeToken.safeTransfer(_treasury, treasuryAmount);
+        _arcadeToken.safeTransfer(_treasury, treasuryAmount);

-        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);
+        emit Distribute(address(_arcadeToken), _treasury, treasuryAmount);
     }
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L89-L98

### [G-03-2]Function `toDevPartner` Cache `arcadeToken` saves 105 Gas on average

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | -    | -   | 64678 | 
| After  | -    | -   | 64573 | 

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
89:    function toDevPartner(address _devPartner) external onlyOwner {

95:        arcadeToken.safeTransfer(_devPartner, devPartnerAmount);

97:        emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);
98:    }
```

```diff
--- a/contracts/token/ArcadeTokenDistributor.sol
+++ b/contracts/token/ArcadeTokenDistributor.sol
@@ -91,10 +91,11 @@ contract ArcadeTokenDistributor is Ownable {
         if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");

         devPartnerSent = true;
+         IArcadeToken  _arcadeToken = arcadeToken;

-        arcadeToken.safeTransfer(_devPartner, devPartnerAmount);
+        _arcadeToken.safeTransfer(_devPartner, devPartnerAmount);

-        emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);
+        emit Distribute(address(_arcadeToken), _devPartner, devPartnerAmount);
     }

```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L105-L114

### [G-03-3]Function `toCommunityRewards()` cache `arcadeToken` to save 124 Gas on average

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | -    | -   | 64688 | 
| After  | 64560    | 64572   | 64564 | 

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
105:    function toCommunityRewards(address _communityRewards) external onlyOwner {

111:        arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);

113:        emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);
114:    }
```

```diff
--- a/contracts/token/ArcadeTokenDistributor.sol
+++ b/contracts/token/ArcadeTokenDistributor.sol
@@ -107,10 +107,11 @@ contract ArcadeTokenDistributor is Ownable {
         if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");

         communityRewardsSent = true;
+        IArcadeToken  _arcadeToken = arcadeToken;

-        arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);
+        _arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);

-        emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);
+        emit Distribute(address(_arcadeToken), _communityRewards, communityRewardsAmount);
     }
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L121-L130

### [G-03-4]Function `toCommunityAirdrop()`: Cache `arcadeToken` to save 116 Gas on average

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | -    | -   | 64710 | 
| After  | -    | -   | 64594 | 

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
121:    function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {

127:        arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);

129:        emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);
130:    }
```

```diff
--- a/contracts/token/ArcadeTokenDistributor.sol
+++ b/contracts/token/ArcadeTokenDistributor.sol
@@ -123,10 +123,11 @@ contract ArcadeTokenDistributor is Ownable {
         if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");

         communityAirdropSent = true;
+        IArcadeToken  _arcadeToken = arcadeToken;

-        arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);
+        _arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);

-        emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);
+        emit Distribute(address(_arcadeToken), _communityAirdrop, communityAirdropAmount);
     }
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L138-L147

### [G-03-5]Function `toTeamVesting()`: cache `arcadeToken` to save 116 Gas on average on average

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | 47610    | 64710   | 48815 | 
| After  | 47494    | 64594   | 48699 | 

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
138:    function toTeamVesting(address _vestingTeam) external onlyOwner {

144:        arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);

146:        emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);
147:    }
```

```diff
--- a/contracts/token/ArcadeTokenDistributor.sol
+++ b/contracts/token/ArcadeTokenDistributor.sol
@@ -140,10 +140,11 @@ contract ArcadeTokenDistributor is Ownable {
         if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");

         vestingTeamSent = true;
+        IArcadeToken  _arcadeToken = arcadeToken;

-        arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);
+        _arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);

-        emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);
+        emit Distribute(address(_arcadeToken), _vestingTeam, vestingTeamAmount);
     }

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L155-L164

### [G-03-6]Function `toPartnerVesting()` : cache `arcadeToken` to save 116 Gas on average 

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | 59887    | 64687   | 64663 | 
| After  | 59771    | 64571   | 64547 | 

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
155:    function toPartnerVesting(address _vestingPartner) external onlyOwner {

161:        arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);

163:        emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);
164:    }
```


```diff
@@ -157,10 +157,11 @@ contract ArcadeTokenDistributor is Ownable {
         if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");

         vestingPartnerSent = true;
+        IArcadeToken  _arcadeToken = arcadeToken;

-        arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);
+        _arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);

-        emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);
+        emit Distribute(address(_arcadeToken), _vestingPartner, vestingPartnerAmount);
     }

```

## [G-04]We can make the following functions more optimal

### Function `getMultiplier()` can be benefit from some refactoring

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L418-L427
```solidity
File: /contracts/NFTBoostVault.sol
418:    function getMultiplier(address tokenAddress, uint128 tokenId) public view override returns (uint128) {
419:        NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];

422:        if (tokenAddress == address(0) || tokenId == 0) {
423:            return 1e3;
424:        }

426:        return multiplierData.multiplier;
427:    }
```
If the user does not specify a ERC1155 nft, there is no need to load `multiplierData` here `_getMultipliers()[tokenAddress][tokenId];` since we know the `multiplierData` is a constant number `1e3`
```diff
diff --git a/contracts/NFTBoostVault.sol b/contracts/NFTBoostVault.sol
index 5f907ee..3f45c0f 100644
--- a/contracts/NFTBoostVault.sol
+++ b/contracts/NFTBoostVault.sol
@@ -416,12 +416,11 @@ contract NFTBoostVault is INFTBoostVault, BaseVotingVault {
      * @return                          The token multiplier.
      */
     function getMultiplier(address tokenAddress, uint128 tokenId) public view override returns (uint128) {
-        NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];
-
         // if a user does not specify a ERC1155 nft, their multiplier is set to 1
         if (tokenAddress == address(0) || tokenId == 0) {
             return 1e3;
         }
+        NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];

         return multiplierData.multiplier;
     }
```


## [G-05]Inline the modifier for optimal checks

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L167-L170
```solidity
File: /contracts/token/ArcadeToken.sol
167:    modifier onlyMinter() {
168:        if (msg.sender != minter) revert AT_MinterNotCaller(minter);
169:        _;
170:    }
```

The above modifier only checks if `msg.sender != minter` and reverts if they are not equal.
The modifier is called on https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132 and https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L145


The first function `setMinter` has the following implementation
```solidity
File: /contracts/token/ArcadeToken.sol
132:    function setMinter(address _newMinter) external onlyMinter {
133:        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");
```

Due to the use of `onlyMinter` the compiler would interpret this as
```solidity
File: /contracts/token/ArcadeToken.sol
    function setMinter(address _newMinter) external{
			if (msg.sender != minter) revert AT_MinterNotCaller(minter);
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");
```

Now note, the check from the modifier `msg.sender != minter` involves reading a state variable `minter` and reverting if not equal to msg.sender.
The check from the function itself only reads a function parameter, way cheaper compared to the check from the modifier. As it is if the modifier check passes but we revert on the function parameter check `_newMinter == address(0)` we would waste the gas used to read the state variable `minter`. If we inline the modifier ourselves, we can reorder the checks to have the state reading check come after the function parameter reading check

```diff
-    function setMinter(address _newMinter) external onlyMinter {
+    function setMinter(address _newMinter) external  {
         if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");
+        if (msg.sender != minter) revert AT_MinterNotCaller(minter);

         minter = _newMinter;
         emit MinterUpdated(minter);
```

Similar thing applies to the function `mint` which can be refactored as follows

```diff
-    function mint(address _to, uint256 _amount) external onlyMinter {
-        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
+    function mint(address _to, uint256 _amount) external  {
         if (_to == address(0)) revert AT_ZeroAddress("to");
         if (_amount == 0) revert AT_ZeroMintAmount();
+        if (msg.sender != minter) revert AT_MinterNotCaller(minter);
+        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);

```

## [G-06]Emitting storage values instead of the memory one.

Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132-L137

### Emit `_newMinter` instead of `minter`

```solidity
File: /contracts/token/ArcadeToken.sol
132:    function setMinter(address _newMinter) external onlyMinter {
133:        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

135:        minter = _newMinter;
136:        emit MinterUpdated(minter);
137:    }
```

```diff
diff --git a/contracts/token/ArcadeToken.sol b/contracts/token/ArcadeToken.sol
index 8cbf058..6297773 100644
--- a/contracts/token/ArcadeToken.sol
+++ b/contracts/token/ArcadeToken.sol
@@ -133,7 +133,7 @@ contract ArcadeToken is ERC20, ERC20Burnable, IArcadeToken, ERC20Permit {
         if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

         minter = _newMinter;
-        emit MinterUpdated(minter);
+        emit MinterUpdated(_newMinter);
     }
```

## [G-07]Optimizing check order for cost efficient function execution

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L145-L148

### [G-07-1]Avoid reading state variables if we can revert on function params

```solidity
File: /contracts/token/ArcadeToken.sol
145:    function mint(address _to, uint256 _amount) external onlyMinter {
146:        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
147:        if (_to == address(0)) revert AT_ZeroAddress("to");
148:        if (_amount == 0) revert AT_ZeroMintAmount();
```
The first check reads a state variable `mintingAllowedAfter` then reverts if it's less than `block.timestamp`. The next two checks just read function parameters `_to and _amount` and also revert in case any of them is zero. If the first check passes but the second fails, we would end up reverting after spending gas doing a state read. As it is cheaper to read function parameters, we should reorder the checks here to validate parameters first before making any state reads
```diff
     function mint(address _to, uint256 _amount) external onlyMinter {
-        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
         if (_to == address(0)) revert AT_ZeroAddress("to");
         if (_amount == 0) revert AT_ZeroMintAmount();
+        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);

```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L73-L75

### [G-07-2]Validate function parameters before reading any state variable

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
73:    function toTreasury(address _treasury) external onlyOwner {
74:        if (treasurySent) revert AT_AlreadySent();
75:        if (_treasury == address(0)) revert AT_ZeroAddress("treasury");
```

```diff
     function toTreasury(address _treasury) external onlyOwner {
-        if (treasurySent) revert AT_AlreadySent();
         if (_treasury == address(0)) revert AT_ZeroAddress("treasury");
+        if (treasurySent) revert AT_AlreadySent();
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L89-L91

### [G-07-3]Let's validate function parameter `_devPartner` before trying to read the state variable `devPartnerSent`

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
89:    function toDevPartner(address _devPartner) external onlyOwner {
90:        if (devPartnerSent) revert AT_AlreadySent();
91:        if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");
```

```diff
     function toDevPartner(address _devPartner) external onlyOwner {
-        if (devPartnerSent) revert AT_AlreadySent();
         if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");
+        if (devPartnerSent) revert AT_AlreadySent();
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L105-L107

### [G-07-4]Checks should be reordered to validate function parameters first

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
105:    function toCommunityRewards(address _communityRewards) external onlyOwner {
106:        if (communityRewardsSent) revert AT_AlreadySent();
107:        if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");
```

```diff
     function toCommunityRewards(address _communityRewards) external onlyOwner {
-        if (communityRewardsSent) revert AT_AlreadySent();
         if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");
+        if (communityRewardsSent) revert AT_AlreadySent();

```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L121-L123

### [G-07-5]Reorder the checks to validate the function parameter first

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
121:    function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {
122:        if (communityAirdropSent) revert AT_AlreadySent();
123:        if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");
```

```diff
     function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {
-        if (communityAirdropSent) revert AT_AlreadySent();
         if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");
+        if (communityAirdropSent) revert AT_AlreadySent();

```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L138-L140

### [G-07-6]We should check the validity of `_vestingTeam` before reading from state

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
138:    function toTeamVesting(address _vestingTeam) external onlyOwner {
139:        if (vestingTeamSent) revert AT_AlreadySent();
140:        if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");
```

```diff
     function toTeamVesting(address _vestingTeam) external onlyOwner {
-        if (vestingTeamSent) revert AT_AlreadySent();
         if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");
-
+        if (vestingTeamSent) revert AT_AlreadySent();
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L155-L157

### [G-07-7]Validate the parameter `_vestingPartner` before checking the state variable

```solidity
File: /contracts/token/ArcadeTokenDistributor.sol
155:    function toPartnerVesting(address _vestingPartner) external onlyOwner {
156:        if (vestingPartnerSent) revert AT_AlreadySent();
157:        if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");
```

```diff
     function toPartnerVesting(address _vestingPartner) external onlyOwner {
-        if (vestingPartnerSent) revert AT_AlreadySent();
         if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");
-
+        if (vestingPartnerSent) revert AT_AlreadySent();
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L98-L109

### [G-07-8]Validate on the fly (no need to load all states then validate later - load while validating)

```solidity
File: /contracts/nft/ReputationBadge.sol
98:    function mint(

104:    ) external payable {
105:        uint256 mintPrice = mintPrices[tokenId] * amount;
106:        uint48 claimExpiration = claimExpirations[tokenId];

108:        if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));
109:        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
```

```diff
diff --git a/contracts/nft/ReputationBadge.sol b/contracts/nft/ReputationBadge.sol
index 7375cea..e95a58f 100644
--- a/contracts/nft/ReputationBadge.sol
+++ b/contracts/nft/ReputationBadge.sol
@@ -103,10 +103,10 @@ contract ReputationBadge is ERC1155, AccessControl, ERC1155Burnable, IReputation
         bytes32[] calldata merkleProof
     ) external payable {
         uint256 mintPrice = mintPrices[tokenId] * amount;
+        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
         uint48 claimExpiration = claimExpirations[tokenId];
-
         if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));
-        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
+
         if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
         if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
             revert RB_InvalidClaimAmount(amount, totalClaimable);
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L223-L225

### [G-07-9]Validate functional parameter before making the function call

```solidity
File: /contracts/NFTBoostVault.sol
223:    function withdraw(uint128 amount) external override nonReentrant {
224:        if (getIsLocked() == 1) revert NBV_Locked();
225:        if (amount == 0) revert NBV_ZeroAmount();
```
The first if statements calls the function `getIsLocked()` which reads from storage and then compares the result with 1. If they are equal we revert. The second if statement checks if a function parameter `amount` is equal to `0` and reverts if true. As the second check is cheaper and we revert on both checks, we should perform the cheaper check first.
```diff
diff --git a/contracts/NFTBoostVault.sol b/contracts/NFTBoostVault.sol
index 5f907ee..f6e5a7f 100644
--- a/contracts/NFTBoostVault.sol
+++ b/contracts/NFTBoostVault.sol
@@ -221,8 +221,8 @@ contract NFTBoostVault is INFTBoostVault, BaseVotingVault {
      * @param amount                      The amount of token to withdraw.
      */
     function withdraw(uint128 amount) external override nonReentrant {
-        if (getIsLocked() == 1) revert NBV_Locked();
         if (amount == 0) revert NBV_ZeroAmount();
+        if (getIsLocked() == 1) revert NBV_Locked();
```


## [G-08]Don't cache if used once

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L204-L214
```solidity
File: /contracts/nft/ReputationBadge.sol
204:    function _verifyClaim(

209:    ) internal view returns (bool) {
210:        bytes32 rewardsRoot = claimRoots[tokenId];
211:        bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));

213:        return MerkleProof.verify(merkleProof, rewardsRoot, leafHash);
214:    }
```
In the above function, the variables `rewardsRoot` and `leafHash` are only being used. As such we can replace their occurrence in the parts they are being used by the actual operation as shown below. Unless it doesn't look good for readability purposes, we can refactor as below
```diff
@@ -207,10 +207,9 @@ contract ReputationBadge is ERC1155, AccessControl, ERC1155Burnable, IReputation
         uint256 totalClaimable,
         bytes32[] calldata merkleProof
     ) internal view returns (bool) {
-        bytes32 rewardsRoot = claimRoots[tokenId];
-        bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));
-
-        return MerkleProof.verify(merkleProof, rewardsRoot, leafHash);
+        // rewardsRoot = claimRoots[tokenId]
+        // leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable))
+        return MerkleProof.verify(merkleProof, claimRoots[tokenId],  keccak256(abi.encodePacked(recipient, tokenId, totalClaimable)));
     }
```

## [G-09]Result of function call should be cached rather than call the function more than once

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L154-L157

### We can cache the results of `totalSupply()` Sad path only

```solidity
File: /contracts/token/ArcadeToken.sol
154:        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
155:        if (_amount > mintCapAmount) {
156:            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
157:        }
```

```diff
         // inflation cap enforcement - 2% of total supply
-        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
+        uint256 _totalSupply = totalSupply();
+        uint256 mintCapAmount = (_totalSupply * MINT_CAP) / PERCENT_DENOMINATOR;
         if (_amount > mintCapAmount) {
-            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
+            revert AT_MintingCapExceeded(_totalSupply, mintCapAmount, _amount);
         }
```

## [G-10]Nested if is cheaper than single statement

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L317-L320
```solidity
File: /contracts/NFTBoostVault.sol
317:        if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
318:            // withdraw the current ERC1155 from the registration
319:            _withdrawNft();
320:        }
```

```diff
-        if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
-            // withdraw the current ERC1155 from the registration
-            _withdrawNft();
+        if (registration.tokenAddress != address(0)) {
+            if (registration.tokenId != 0) {
+                // withdraw the current ERC1155 from the registration
+                _withdrawNft();
+            }
         }
```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L632-L634
```solidity
File: /contracts/NFTBoostVault.sol
632:        if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
633:            return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;
634:        }
```

```diff
diff --git a/contracts/NFTBoostVault.sol b/contracts/NFTBoostVault.sol
index 5f907ee..eff4708 100644
--- a/contracts/NFTBoostVault.sol
+++ b/contracts/NFTBoostVault.sol
@@ -629,8 +629,10 @@ contract NFTBoostVault is INFTBoostVault, BaseVotingVault {
     ) internal view virtual returns (uint256) {
         uint128 locked = registration.amount - registration.withdrawn;

-        if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
-            return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;
+        if (registration.tokenAddress != address(0) ) {
+             if ( registration.tokenId != 0) {
+                return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;
+             }
         }

```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L659-L661
```solidity
File: /contracts/NFTBoostVault.sol
659:        if (tokenAddress != address(0) && tokenId != 0) {
660:            _lockNft(from, tokenAddress, tokenId, nftAmount);
661:        }
```

```diff

-        if (tokenAddress != address(0) && tokenId != 0) {
-            _lockNft(from, tokenAddress, tokenId, nftAmount);
+        if (tokenAddress != address(0)){
+            if (tokenId != 0) {
+                _lockNft(from, tokenAddress, tokenId, nftAmount);
+            }
         }
     }
```

## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
