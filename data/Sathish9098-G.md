# GAS OPTIMIZATIONS

##

## [G-1] State variables can be packed to use fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas)

### ``ArcadeToken.sol`` : ``minter``,``mintingAllowedAfter`` can be packed together : Saves ``2000 GAS``, ``1 SLOT``

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L93-L97

``mintingAllowedAfter``can be uint96 since this always stores the epoch time. This means that a ``uint96`` can store timestamps for up to ``2^32-1 years``, which is approximately ``2147483648 years``. The Unix epoch is the number of seconds since January 1, 1970, 00:00:00 UTC. There are approximately 2^64 seconds in a year, so a uint96 can store timestamps for approximately 2^32 years without overflow.


```diff
FILE: 2023-07-arcade/contracts/token/ArcadeToken.sol

93:  /// @notice Minter contract address responsible for minting future tokens
94:    address public minter;
95:
96:    /// @notice The timestamp after which minting may occur
- 97:    uint256 public mintingAllowedAfter;
+ 97:    uint96 public mintingAllowedAfter;

```
##

## [G-3] State variables should be cached in stack variables rather than re-reading them from storage

### Saves ``3000 GAS``, ``30 SLODs``

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

### ``mintingAllowedAfter`` should be cached avoid ``1 SLOD`` ,`` 100 GAS``

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L146


```diff
FILE: Breadcrumbs2023-07-arcade/contracts/token/ArcadeToken.sol

+ uint256 mintingAllowedAfter_ = mintingAllowedAfter;
- 146: if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
+ 146: if (block.timestamp < mintingAllowedAfter_ ) revert AT_MintingNotStarted(mintingAllowedAfter_ , block.timestamp);
147:  if (_to == address(0)) revert AT_ZeroAddress("to");

```

### ``unassigned.data, grant.allocation, grant.cliff, grant.delegatee, grant.latestVotingPower`` can be cached : Saves ``1500 GAS, 15 SLODs``

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L114-L148

```diff
FILE: 2023-07-arcade/contracts/ARCDVestingVault.sol

Saves 4 SLODs

        Storage.Uint256 storage unassigned = _unassigned();
+      uint256 _data = unassigned.data ;
-        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
+        if (_data  < amount) revert AVV_InsufficientBalance(_data);

        // load the grant
        ARCDVestingVaultStorage.Grant storage grant = _grants()[who];

        // if this address already has a grant, a different address must be provided
        // topping up or editing active grants is not supported.
        if (grant.allocation != 0) revert AVV_HasGrant();

        // load the delegate. Defaults to the grant owner
        delegatee = delegatee == address(0) ? who : delegatee;

        // calculate the voting power. Assumes all voting power is initially locked.
        uint128 newVotingPower = amount;

        // set the new grant
        grant.allocation = amount;
        grant.cliffAmount = cliffAmount;
        grant.withdrawn = 0;
        grant.created = startTime;
        grant.expiration = expiration;
        grant.cliff = cliff;
        grant.latestVotingPower = newVotingPower;
        grant.delegatee = delegatee;

        // update the amount of unassigned tokens
-        unassigned.data -= amount;
+        unassigned.data =_data + amount;
        // update the delegatee's voting power
        History.HistoricalBalances memory votingPower = _votingPower();
-        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
+        uint256 delegateeVotes = votingPower.loadTop(delegatee);
-        votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);
+        votingPower.push(delegatee, delegateeVotes + newVotingPower);
        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));

Saves 1 SLOD

+  uint128 _allocation = grant.allocation ;
-  if (grant.allocation == 0) revert AVV_NoGrantSet();
+  if (_allocation  == 0) revert AVV_NoGrantSet();
        // get the amount of withdrawable tokens
        uint256 withdrawable = _getWithdrawableAmount(grant);
        grant.withdrawn += uint128(withdrawable);
        token.safeTransfer(who, withdrawable);

        // transfer the remaining tokens to the vesting manager
-        uint256 remaining = grant.allocation - grant.withdrawn;
+        uint256 remaining = _allocation  - grant.withdrawn;

Saves 2 SLODs

212:   Storage.Uint256 storage unassigned = _unassigned();
+   uint256 data_= unassigned.data;
- 213: if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
+ 213: if (data_ < amount) revert AVV_InsufficientBalance(data_);
214:        // update unassigned value
- 215:        unassigned.data -= amount;
+ 215:        unassigned.data =data_- amount;

Saves 1 SLOD

+  uint128 cliff_ =  grant.cliff ;
- 234:   if (grant.cliff > block.number) revert AVV_CliffNotReached(grant.cliff);
+ 234:   if (cliff_ > block.number) revert AVV_CliffNotReached(cliff_);

Saves 5 SLODs

  ARCDVestingVaultStorage.Grant storage grant = _grants()[msg.sender];
+  address delegatee_= grant.delegatee;
-        if (to == grant.delegatee) revert AVV_AlreadyDelegated();
+        if (to == delegatee_) revert AVV_AlreadyDelegated();

        History.HistoricalBalances memory votingPower = _votingPower();
-        uint256 oldDelegateeVotes = votingPower.loadTop(grant.delegatee);
+        uint256 oldDelegateeVotes = votingPower.loadTop(delegatee_);

        // Remove old delegatee's voting power and emit event
+    uint256 latestVotingPower_= grant.latestVotingPower;
-        votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);
+        votingPower.push(delegatee_, oldDelegateeVotes - latestVotingPower_);
-        emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));
+        emit VoteChange(delegatee_, msg.sender, -1 * int256(latestVotingPower_));

        // Note - It is important that this is loaded here and not before the previous state change because if
        // to == grant.delegatee and re-delegation was allowed we could be working with out of date state.
        uint256 newDelegateeVotes = votingPower.loadTop(to);

        // add voting power to the target delegatee and emit event
-         votingPower.push(to, newDelegateeVotes + grant.latestVotingPower);
+         votingPower.push(to, newDelegateeVotes + latestVotingPower_);
        // update grant delgatee info
        grant.delegatee = to;

-        emit VoteChange(to, msg.sender, int256(uint256(grant.latestVotingPower)));
+        emit VoteChange(to, msg.sender, int256(uint256(latestVotingPower_)));

Saves 1 SLOD
 
+ address delegatee_= grant.delegatee;
-  uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
+  uint256 delegateeVotes = votingPower.loadTop(delegatee_);

        uint256 newVotingPower = grant.allocation - grant.withdrawn;

        // get the change in voting power. voting power can only go down
        // since the sync is only called when tokens are claimed or grant revoked
        int256 change = int256(newVotingPower) - int256(grant.latestVotingPower);
        // we multiply by -1 to avoid underflow when casting
-        votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));
+        votingPower.push(delegatee_, delegateeVotes - uint256(change * -1));
        grant.latestVotingPower = newVotingPower;

-        emit VoteChange(grant.delegatee, who, change);
+        emit VoteChange(delegatee_, who, change);

```

### ``registration.delegatee,registration.latestVotingPower,balance.data,registration.tokenId,`` should be cached : Saves ``1200 GAS, 12 SLOD``

```diff
FILE: Breadcrumbs2023-07-arcade/contracts/NFTBoostVault.sol

Saves 1 SLOD

+ address delegatee_ = registration.delegatee ;
- 152: if (registration.delegatee == address(0)) {
+ 152: if (delegatee_ == address(0)) {
153:            _registerAndDelegate(user, amount, 0, address(0), delegatee);
154:        } else {
155:            // if user supplies new delegatee address revert
- 156:            if (delegatee != registration.delegatee) revert NBV_WrongDelegatee(delegatee, registration.delegatee);
+ 156:  if (delegatee != delegatee_) revert NBV_WrongDelegatee(delegatee, delegatee_);

Saves 3 SLODs


185: NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];
186:
187:        // If to address is already the delegate, don't send the tx
+  address delegatee_ = registration.delegatee;
- 188:        if (to == registration.delegatee) revert NBV_AlreadyDelegated();
+ 188:        if (to == delegatee_ ) revert NBV_AlreadyDelegated();
189:
190:        History.HistoricalBalances memory votingPower = _votingPower();
- 191:        uint256 oldDelegateeVotes = votingPower.loadTop(registration.delegatee);
+ 191:        uint256 oldDelegateeVotes = votingPower.loadTop(delegatee_ );
192:
193:        // Remove voting power from old delegatee and emit event
+ uint128 latestVotingPower_ = registration.latestVotingPower;
- 194:        votingPower.push(registration.delegatee, oldDelegateeVotes - registration.latestVotingPower);
+ 194:        votingPower.push(delegatee_ , oldDelegateeVotes - latestVotingPower_ );
- 195:        emit VoteChange(msg.sender, registration.delegatee, -1 * int256(uint256(registration.latestVotingPower)));
+ 195:        emit VoteChange(msg.sender, delegatee_ , -1 * int256(uint256(latestVotingPower_ )));

Saves 1 SLOD

231: Storage.Uint256 storage balance = _balance();
+ uint256 data_=balance.data;
- 232: if (balance.data < amount) revert NBV_InsufficientBalance();
+ 232: if (data_ < amount) revert NBV_InsufficientBalance();
233:
234:        // get the withdrawable amount
235:        uint256 withdrawable = _getWithdrawableAmount(registration);
236:        if (withdrawable < amount) revert NBV_InsufficientWithdrawableBalance(withdrawable);
237:
238:        // update contract balance
- 239:        balance.data -= amount;
+ 239:        balance.data =data_- amount;

Saves 4 SLODs

+  address tokenAddress_ = registration.tokenAddress;
+  uint128 tokenId_ = registration.tokenId ; 
- 552:        if (registration.tokenAddress == address(0) || registration.tokenId == 0)
+ 552:        if (tokenAddress_  == address(0) || tokenId_  == 0)
- 553:            revert NBV_InvalidNft(registration.tokenAddress, registration.tokenId);
+ 553:            revert NBV_InvalidNft(tokenAddress_ , tokenId_ );
554:
555:        // transfer ERC1155 back to the user
- 556:        IERC1155(registration.tokenAddress).safeTransferFrom(
+ 556:        IERC1155(tokenAddress_ ).safeTransferFrom(
557:            address(this),
558:            msg.sender,
- 559:            registration.tokenId,
+ 559:            tokenId_ ,
560:            1,
561:            bytes("")
562:        );

Saves 3 SLODs

+ address delegatee_=registration.delegatee;
- uint256 delegateeVotes = votingPower.loadTop(registration.delegatee);
+ uint256 delegateeVotes = votingPower.loadTop(delegatee_);

        uint256 newVotingPower = _currentVotingPower(registration);
        // get the change in voting power. Negative if the voting power is reduced
        int256 change = int256(newVotingPower) - int256(uint256(registration.latestVotingPower));

        // do nothing if there is no change
        if (change == 0) return;
        if (change > 0) {
-            votingPower.push(registration.delegatee, delegateeVotes + uint256(change));
+            votingPower.push(delegatee_, delegateeVotes + uint256(change));
        } else {
            // if the change is negative, we multiply by -1 to avoid underflow when casting
-             votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));
+             votingPower.push(delegatee_, delegateeVotes - uint256(change * -1));
        }

        registration.latestVotingPower = uint128(newVotingPower);

-        emit VoteChange(who, registration.delegatee, change);
+        emit VoteChange(who, delegatee_, change);

```

### ``mintingAllowedAfter`` should be cached : Saves ``100 GAS,1 SLOD``

```diff
FILE: 2023-07-arcade/contracts/token/ArcadeToken.sol

Saves 1 SLOD

+ uint256 mintingAllowedAfter_ = mintingAllowedAfter;
- 146: if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
+ 146: if (block.timestamp < mintingAllowedAfter_ ) revert AT_MintingNotStarted(mintingAllowedAfter_ , block.timestamp);

```

### ``baseURI`` should be cached  : Saves ``100 GAS, 1 SLOD``

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/BadgeDescriptor.sol#L49

```diff
FILE: Breadcrumbs2023-07-arcade/contracts/nft/BadgeDescriptor.sol

+ string baseURI_ = baseURI;
- 49:  return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
+ 49:  return bytes(baseURI_ ).length > 0 ? string(abi.encodePacked(baseURI_, tokenId.toString())) : "";

```

##

## [G-2] Don't emit state variable when stack variable available

### Saves ``200 GAS``, ``2 SLODs ``

If stack can be emitted when ever available to save gas 

### ``_newMinter`` can be used instead of ``minter`` state variable : Saves ``100 GAS``,``1 SLOD``

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L136

```diff
FILE: Breadcrumbs2023-07-arcade/contracts/token/ArcadeToken.sol

135:   minter = _newMinter;
- 136:   emit MinterUpdated(minter);
+ 136:   emit MinterUpdated(_newMinter);

```
### ``delegatee`` can be used instead of ``grant.delegatee`` state variable : Saves ``100 GAS``,``1 SLOD``

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L148



```diff
FILE: Breadcrumbs2023-07-arcade/contracts/ARCDVestingVault.sol

138: grant.delegatee = delegatee;

- 148: emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
+ 148: emit VoteChange(delegatee, who, int256(uint256(newVotingPower)));

```

##

## [G-]  Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata

### ``lastAllowanceSet[token]`` should be cached : Saves ``100 GAS``, ``1 SLOT``


https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L308-L310

```diff
FILE: 2023-07-arcade/contracts/ArcadeTreasury.sol

Saves 100 GAS,1 SLOD

307: // enforce cool down period
+ uint48 _token = lastAllowanceSet[token];
- 308:        if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {
+ 308:        if (uint48(block.timestamp) < _token  + SET_ALLOWANCE_COOL_DOWN) {
- 309:            revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);
+ 309:            revert T_CoolDownPeriod(block.timestamp, _token  + SET_ALLOWANCE_COOL_DOWN);
310:        }

```

### ``amountClaimed[recipient][tokenId]`` should be cached : Saves ``100 GAS, 1 SLOD``

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L111-L116

```diff
FILE: Breadcrumbs2023-07-arcade/contracts/nft/ReputationBadge.sol

 uint48 claimExpiration = claimExpirations[tokenId];
+  uint256 recipienttokenId = amountClaimed[recipient][tokenId] ;
        if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));
        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
-         if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
+         if (recipienttokenId + amount > totalClaimable) {
            revert RB_InvalidClaimAmount(amount, totalClaimable);
        }

        // increment amount claimed
-         amountClaimed[recipient][tokenId] += amount;
+         amountClaimed[recipient][tokenId] = recipienttokenId + amount;


```

##

## <x> += <y> costs more gas than <x> = <x> + <y> for state variables (<x> -= <y> )

### Saves ``1695 GAS``, ``15 Instances``

Using the addition operator instead of plus-equals saves ``113 gas``

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L117

```diff
FILE: Breadcrumbs2023-07-arcade/contracts/ArcadeTreasury.sol

- 117: gscAllowance[token] -= amount;
+ 117: gscAllowance[token] = gscAllowance[token]- amount;

- 198: gscAllowance[token] -= amount;
+ 198: gscAllowance[token] = gscAllowance[token] - amount;

```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L166

```diff
FILE: Breadcrumbs2023-07-arcade/contracts/ARCDVestingVault.sol

- 166: grant.withdrawn += uint128(withdrawable);
+ 166: grant.withdrawn =grant.withdrawn + uint128(withdrawable);

- 171: grant.withdrawn += uint128(remaining);
+ 171: grant.withdrawn =grant.withdrawn + uint128(remaining);

- 200: unassigned.data += amount;
+ 200: unassigned.data = unassigned.data + amount;

- 242: grant.withdrawn += uint128(withdrawable);
+ 242: grant.withdrawn =grant.withdrawn + uint128(withdrawable);

- 244: grant.withdrawn += uint128(amount);
+ 244: grant.withdrawn =grant.withdrawn + uint128(amount);

```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L116


```diff
FILE: 2023-07-arcade/contracts/nft/ReputationBadge.sol

- 116: amountClaimed[recipient][tokenId] += amount;
+ 116: amountClaimed[recipient][tokenId] =amountClaimed[recipient][tokenId] + amount;

```

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L161

```diff
FILE: Breadcrumbs2023-07-arcade/contracts/NFTBoostVault.sol

- 161:  balance.data += amount;
+ 161:  balance.data =balance.data + amount;

- 164: registration.amount += amount;
+ 164: registration.amount =registration.amount + amount;

- 239: balance.data -= amount;
+ 239: balance.data =balance.data - amount;

- 241: registration.withdrawn += amount;
+ 241: registration.withdrawn =registration.withdrawn + amount;

- 278: balance.data += amount;
+ 278: balance.data =balance.data + amount;

- 281: registration.amount += amount;
+ 281: registration.amount =registration.amount +  amount;


```
##

## [G-] IF’s/require() statements that check input arguments should be at the top of the function

FAIL CHEEPLY INSTEAD OF COSTLY

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Function parameters should be checked before state variables. If any revert after state variable check this will cost 


```diff
FILE: 2023-07-arcade/contracts/token/ArcadeTokenDistributor.sol



```

