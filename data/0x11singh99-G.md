### Gas Optimizations List

| Number | Optimization Details                                                                                                     | Instances |
| :----: | :----------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Do not update storage variable with default value zero to zero,saves Gsreset.                                            |     1     |
| [G-02] | Do not calculate constant variables                                                                                      |     6     |
| [G-03] | State variables used in a function should be cached in stack variables rather than re-reading them from storage.         |    18     |
| [G-04] | Function calls can be cached rather than re calling from same function with same value will save gas                     |     1     |
| [G-05] | Use assembly for `address(0)` comparison to save gas                                                                     |     3     |
| [G-06] | \<x> += \<y> / \<x> -= \<y> costs more gas than \<x> = \<x> + \<y> / \<x> = \<x> - \<y> for state variables              |    16     |
| [G-07] | Using ternary operator instead of if else saves gas                                                                      |     2     |
| [G-08] | Use `delete` for state variables to reinitialize with their default value rather than assigning default value saves gas. |    12     |
| [G-09] | Use unchecked{} whenever underflow/overflow not possible                                                                 |     2     |
| [G-10] | Use nested if and, avoid multiple check combinations using &&                                                            |     5     |

Total 10 issues.

## [G-01] Do not update storage variable with default value zero to zero,saves Gsreset

It saves Gsreset (2900 Gas) , Paid for an SSTORE operation when the storage value’s zeroness remains unchanged or is set
to zero.

_Total 1 instance - 1 file:_

### Instance#1 : Function `addGrantAndDelegate` adding new grants,updating not supported. So `grant.withdrawn` is already 0 , no need to update to 0 again.

```solidity
File:    contracts/ARCDVestingVault.sol

118:         ARCDVestingVaultStorage.Grant storage grant = _grants()[who];

        // if this address already has a grant, a different address must be provided
        // topping up or editing active grants is not supported.
122:         if (grant.allocation != 0) revert AVV_HasGrant();


133:         grant.withdrawn = 0;//@audit no need to reset to zero. It is by default for new grants
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L118-122

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L133

## [G-02] Do not calculate constant variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant
variable is recomputed each time that the variable is used, which wastes some gas each time of use.

_Total 6 instances - 2 files:_

### Instance#1-3 : Assign direct calculated simple constant value to constant types after calculating this value off chain(remix OR using vscode) rather calculating here in contract at the time of assigning.

```solidity
File : contracts/nft/ReputationBadge.sol

44:    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");
45:    bytes32 public constant BADGE_MANAGER_ROLE = keccak256("BADGE_MANAGER");
46:    bytes32 public constant RESOURCE_MANAGER_ROLE = keccak256("RESOURCE_MANAGER");

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L44C5-L46C83

Recommended Changes : Assign direct hash value and comment these calculating keccak256() functions to show that
assigning hash belongs to which string.

```diff
- 44:    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");
- 45:    bytes32 public constant BADGE_MANAGER_ROLE = keccak256("BADGE_MANAGER");
- 46:    bytes32 public constant RESOURCE_MANAGER_ROLE = keccak256("RESOURCE_MANAGER");

+        //keccak256("ADMIN")
+      bytes32 public constant ADMIN_ROLE = 0xdf8b4c520ffe197c5343c6f5aec59570151ef9a492f2c624fd45ddde6135ec42;
+
+        //keccak256("BADGE_MANAGER");
+      bytes32 public constant BADGE_MANAGER_ROLE = 0x9605363ac419df002bafe0cc1dcbfb19cf2a2e2afa1c7428b115d0be2ecd78e5;
+
+         //keccak256("RESOURCE_MANAGER")
+       bytes32 public constant RESOURCE_MANAGER_ROLE = 0xa08df8e9779f89161e9d4aa6eaffa8e1f95bfbc9b9812ee5dec3c3b19090625d;
```

### Instance#4-6 :

```solidity
File : contracts/ArcadeTreasury.sol

48:    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");
49:    bytes32 public constant GSC_CORE_VOTING_ROLE = keccak256("GSC_CORE_VOTING");
50:    bytes32 public constant CORE_VOTING_ROLE = keccak256("CORE_VOTING");
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L48C5-L50C73

Recommended Changes :Like above Instance#1-3 recommendation, assign direct hash value and comment these calculating
keccak256() functions to show that assigning hash belongs to which string.

## [G‑03] State variables used in a function should be cached in stack variables rather than re-reading them from storage.

The instances below point to the second+ access of a state variable within a function. Caching of a state variable
replaces each Gwarmaccess (100 gas) with a much cheaper stack read

Note: view functions also added since they are used in state changing functions

_18 instances - 5 files:_

### Instance#1: `amountClaimed[recipient][tokenId]` can be cached, saves 1 Gwarmaccess and it's addition with amount also can be cached rather than re-calculating same line in same function saves 1 extra ADD also.

```solidity
File:  contracts/nft/ReputationBadge.sol
111: if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
            revert RB_InvalidClaimAmount(amount, totalClaimable);
        }

        // increment amount claimed
116:        amountClaimed[recipient][tokenId] += amount; //@audit gas re-reading same var. from storage

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L111C9-L116C53

Recommended Changes :

```diff
+        uint256 amountSum = amountClaimed[recipient][tokenId] + amount;
- 111:   if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
+        if ( amountSum > totalClaimable) {
            revert RB_InvalidClaimAmount(amount, totalClaimable);
          }

          // increment amount claimed
- 116:   amountClaimed[recipient][tokenId] += amount;
+        amountClaimed[recipient][tokenId] = amountSum;
```

### Instance#2: Unnecessary reading from storage `minter` while same value available in parameter `_newMinter`.

```solidity
File :  contracts/token/ArcadeToken.sol

132:  function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

            minter = _newMinter;
136:        emit MinterUpdated(minter);//@audit Unnecessary storage read, write _newMinter here instead.
137:    }

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L132C4-L137C6

### Instance#3 : `lastAllowanceSet[token]` can be cached(saves 100 Gas of Gwarmacess) rather than re-reding in same function.

```solidity
File: contracts/ArcadeTreasury.sol

308:      if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {
309:            revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN); //@audit re-reading lastAllowanceSet[token] should be cached rather
310:        }
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L308C7-L310C10

### Instance#4 : Storage variable `unassigned.data` can be cached(saves 200 Gas of Gwarmacess). Inside one function `addGrantAndDelegate`.

```solidity
File: contracts/ARCDVestingVault.sol

114:      Storage.Uint256 storage unassigned = _unassigned();
115:   if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
   ...
141:      unassigned.data -= amount;  //@audit again reading unassigned.data,
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L114C6-L115C87

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L141C7-L141C35

### Instance#5: Unnecessary reading from storage `grant.delegatee` 3 times (line 145-148) while parameter `delegatee` is assigned into this storage variable `grant.delegatee` ,so read from `delegatee` since it is cheap stack read(saves 300 Gas 3 Gwarmaccess) .

```solidity
File:  /contracts/ARCDVestingVault.sol

138:              grant.delegatee = delegatee; //@audit delegatee assigned to grant.delegatee

                  // update the amount of unassigned tokens
                  unassigned.data -= amount;

                 // update the delegatee's voting power
                 History.HistoricalBalances memory votingPower = _votingPower();
145:             uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);//@audit read from delegatee
146:             votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);

148:             emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L138C8-L148C80

### Instance#6-7 : `grant.allocation` and `grant.withdrawn` can be cached rather than re-reading from storage in same function.(saves 3 Gwarmaccess ie. 300 Gas)

Another gas saving here `grant.withdrawn` is updated more than once(line 166 and 172) inside same function so it can be
cached into stack variable from it's already cached stack(So Gas for copy), update into one multiple times(whenever
necessary) and finally write that stack variable into this state variable. (Saves another 100 Gas).

```solidity
File: /contracts/ARCDVestingVault.sol

162:       if (grant.allocation == 0) revert AVV_NoGrantSet(); //@audit gas grant.allocation read

            // get the amount of withdrawable tokens
            uint256 withdrawable = _getWithdrawableAmount(grant);
166:        grant.withdrawn += uint128(withdrawable);//@audit gas grant.withdrawn read
            token.safeTransfer(who, withdrawable);

            // transfer the remaining tokens to the vesting manager
171:        uint256 remaining = grant.allocation - grant.withdrawn; //@audit gas grant.allocation and grant.withdrawn read
172:        grant.withdrawn += uint128(remaining); //@audit gas grant.withdrawn read

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L162C9-L172C1

### Instance#8 : Storage variable `unassigned.data` can be cached(saves 200 Gas of Gwarmacess) rather than reading 3 times in same function.

```solidity
File: /contracts/ARCDVestingVault.sol

212:  Storage.Uint256 storage unassigned = _unassigned();
213:        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);//@audit Gas accessing unassigned.data 2 times
            // update unassigned value
215:        unassigned.data -= amount; //@audit Gas accessing unassigned.data again
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L212C8-L215C35

### Instance#9 : Storage variable `grant.cliff` can be cached(saves 100 Gas of Gwarmacess) rather than reading 2 times in same function.

```solidity
File: /contracts/ARCDVestingVault.sol

234:  if (grant.cliff > block.number) revert AVV_CliffNotReached(grant.cliff);
       //@audit Gas accessing grant.data 2 times. Can be cached.
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L234

### Instance#10-11 : `grant.delegatee` and `grant.latestVotingPower` can be cached rather than re-reading 4 times each from storage in same function.(saves 6 Gwarmaccess ie. 600 Gas)

```solidity
File: /contracts/ARCDVestingVault.sol

260:   function delegate(address to) external {
261:         ARCDVestingVaultStorage.Grant storage grant = _grants()[msg.sender];
262:         if (to == grant.delegatee) revert AVV_AlreadyDelegated();//@audit grant.delegatee read

             History.HistoricalBalances memory votingPower = _votingPower();
265:         uint256 oldDelegateeVotes = votingPower.loadTop(grant.delegatee);//@audit grant.delegatee read

267:         // Remove old delegatee's voting power and emit event
268:         votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);//@audit grant.delegatee and grant.latestVotingPower read
269:         emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));//@audit grant.delegatee and grant.latestVotingPower read

             // Note - It is important that this is loaded here and not before the previous state change because if
             // to == grant.delegatee and re-delegation was allowed we could be working with out of date state.
             uint256 newDelegateeVotes = votingPower.loadTop(to);

             // add voting power to the target delegatee and emit event
276:         votingPower.push(to, newDelegateeVotes + grant.latestVotingPower);//@audit grant.latestVotingPower read

             // update grant delgatee info
             grant.delegatee = to;

281:        emit VoteChange(to, msg.sender, int256(uint256(grant.latestVotingPower)));//@audit  grant.latestVotingPower read
    }
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L260C5-L282C6

### Instance#12 : `grant.delegatee` can be cached rather than re-reading 3 times from storage in same function.(saves 2 Gwarmaccess ie. 200 Gas)

```solidity
File: /contracts/ARCDVestingVault.sol

344:         uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);//@audit grant.delegatee read


352:         votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));//@audit grant.delegatee read

             grant.latestVotingPower = newVotingPower;

356:         emit VoteChange(grant.delegatee, who, change);//@audit grant.delegatee read
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L344C9-L356C55

### Instance#13 : `registration.delegatee` can be cached rather than re-reading. If line 152 condition becomes false then `registration.delegatee` will be accessed total 2-3 times(depends on inner if condition) from storage. So it's better to cache that.(can save 100-200 gas )

```solidity
File: contracts/NFTBoostVault.sol

152:  if (registration.delegatee == address(0)) {
            _registerAndDelegate(user, amount, 0, address(0), delegatee);
        } else {
            // if user supplies new delegatee address revert
156:            if (delegatee != registration.delegatee) revert NBV_WrongDelegatee(delegatee, registration.delegatee);

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L152C8-L156C115

### Instance#14-15 : `registration.delegatee` and `registration.latestVotingPower` can be cached rather than re-reading 4 times and 2 times respectively from storage in same function.(saves 4 Gwarmaccess ie. 400 Gas)

```solidity
File: contracts/NFTBoostVault.sol

188:   if (to == registration.delegatee) revert NBV_AlreadyDelegated();//@audit registration.delegatee read

          History.HistoricalBalances memory votingPower = _votingPower();
191:        uint256 oldDelegateeVotes = votingPower.loadTop(registration.delegatee);//@audit registration.delegatee read

           // Remove voting power from old delegatee and emit event
194:       votingPower.push(registration.delegatee, oldDelegateeVotes - registration.latestVotingPower);//@audit registration.delegatee and registration.latestVotingPower read
195:       emit VoteChange(msg.sender, registration.delegatee, -1 * int256(uint256(registration.latestVotingPower)));//@audit registration.delegatee and registration.latestVotingPower read


```

### Instance#16-17 : `registration.tokenAddress` and `registration.tokenId` can be cached rather than re-reading 2 times each from storage in same function.(can save 2 Gwarmaccess ie. 200 Gas)

```solidity
File: contracts/NFTBoostVault.sol

552:       if (registration.tokenAddress == address(0) || registration.tokenId == 0)
553:            revert NBV_InvalidNft(registration.tokenAddress, registration.tokenId);

             // transfer ERC1155 back to the user
556:        IERC1155(registration.tokenAddress).safeTransferFrom(
            address(this),
            msg.sender,
559:        registration.tokenId,
            1,
            bytes("")
        );
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L552C9-L562C11

### Instance#18 : `registration.delegatee` can be cached rather than re-reading 3 times from storage in same function.(saves 2 Gwarmaccess ie. 200 Gas)

```solidity
581:    uint256 delegateeVotes = votingPower.loadTop(registration.delegatee);//@audit registration.delegatee read

        uint256 newVotingPower = _currentVotingPower(registration);
        // get the change in voting power. Negative if the voting power is reduced
        int256 change = int256(newVotingPower) - int256(uint256(registration.latestVotingPower));

        // do nothing if there is no change
        if (change == 0) return;
        if (change > 0) {
            votingPower.push(registration.delegatee, delegateeVotes + uint256(change));//@audit registration.delegatee read
        } else {
            // if the change is negative, we multiply by -1 to avoid underflow when casting
            votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));//@audit registration.delegatee read
        }

        registration.latestVotingPower = uint128(newVotingPower);

        emit VoteChange(who, registration.delegatee, change);//@audit registration.delegatee read
```

## [G-04] Function calls can be cached rather than re calling from same function with same value will save gas

_1 instance - 1 file:_

### Instance#1 : `totalSupply()` can be cached to save gas instead of re calling in line 156, it will return same value here.

```solidity
File:  contracts/token/ArcadeToken.sol

154:        uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
155:        if (_amount > mintCapAmount) {
156:            revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);
}
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L154C6-L156C81

## [G-05] Use assembly for `address(0)` comparison to save gas

_3 instances - 2 files:_

### Instance#1 :

```solidity
File : contracts/token/ArcadeTokenDistributor.sol

176 : if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();
```

Recommended Changes: Use this OR make a function inside library and write this assembly logic there, call whenever
comparison with address(0) needed. It will make code modular also.

```diff
+       bool isAddressZero;
+        assembly {
+            isAddressZero := eq(address(arcadeToken), 0)
+        }
-    if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();
+    if (!isAddressZero) revert AT_TokenAlreadySet();
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L176

### Instance#2 :

```solidity
File : contracts/NFTBoostVault.sol

247:    if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L247C13-L247C88

Recommended Changes: Change like above Instance#1 recommendation.

### Instance#3:

```solidity
File: contracts/NFTBoostVault.sol

317:  if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L317

Recommended Changes: Change like above Instance#1 recommendation.

## [G-06] \<x> += \<y> / \<x> -= \<y> costs more gas than \<x> = \<x> + \<y> / \<x> = \<x> - \<y> for state variables

The Solidity compiler does not optimize the \<x> += \<y> / \<x> -= \<y> operation for state variables. This means that
every time the state variable is updated, the entire value is copied to memory, the operation is performed, and then the
value is copied back to storage. This is expensive and can be avoided by using \<x> = \<x> + \<y> / \<x> = \<x> - \<y>
instead.

_16 instances - 3 files:_

### Instance#1-2:

```diff
File : /contracts/ArcadeTreasury.sol

- 117:  gscAllowance[token] -= amount;
+ 117:  gscAllowance[token] = gscAllowance[token] - amount;

         ...

- 198:   gscAllowance[token] -= amount;
+ 198:   gscAllowance[token] = gscAllowance[token] - amount;

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L117

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L198

### Instance#3-9:

```diff
File: /contracts/ARCDVestingVault.sol

- 141:    unassigned.data -= amount;
+ 141:    unassigned.data = unassigned.data - amount;

- 166:        grant.withdrawn += uint128(withdrawable);
+ 166:        grant.withdrawn = grant.withdrawn + uint128(withdrawable);

- 172:        grant.withdrawn += uint128(remaining);
+ 172:        grant.withdrawn = grant.withdrawn + uint128(remaining);

- 200:        unassigned.data += amount;
+ 200:        unassigned.data = unassigned.data + amount;

- 215:        unassigned.data -= amount;
+ 215:        unassigned.data = unassigned.data - amount;

- 242:        grant.withdrawn += uint128(withdrawable);
+ 242:        grant.withdrawn = rant.withdrawn + uint128(withdrawable);

- 244:        grant.withdrawn += uint128(amount);
+ 244:        grant.withdrawn = grant.withdrawn + uint128(amount);
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L141

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L166

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L172

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L200

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L215

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L242

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L244

### Instance#10-16:

```solidity
File: /contracts/NFTBoostVault.sol

161:  balance.data += amount;

164: registration.amount += amount;

239: balance.data -= amount;

241: registration.withdrawn += amount;

278: balance.data += amount;

281: registration.amount += amount;

505: balance.data += _amount;
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L161

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L164

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L239

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L241

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L278

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L281

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L505

Recommended Changes: Change like above Instance#1-9 recommendation.

## [G-07] Using ternary operator instead of if else saves gas

When using the if-else construct, Solidity generates bytecode that includes a JUMP operation. The JUMP operation
requires the EVM to change the program counter to a different location in the bytecode, resulting in additional gas
costs. On the other hand, the ternary operator doesn't involve a JUMP operation, as it generates bytecode that utilizes
conditional stack manipulation. The exact amount of gas saved by using the ternary operator instead of the if-else
construct in Solidity can vary depending on the specific scenario, the complexity of the surrounding code, and the
conditions involved

_2 instances - 2 files:_

### Instance#1 :

```solidity
File: /contracts/ArcadeTreasury.sol

365:   if (address(token) == ETH_CONSTANT) {
366:            // will out-of-gas revert if recipient is a contract with logic inside receive()
367:            payable(destination).transfer(amount);
368:        } else {
369:            IERC20(token).safeTransfer(destination, amount);
370:        }
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L365C9-L370C10

Recommended Changes :

```diff
- 365:   if (address(token) == ETH_CONSTANT) {
- 366:            // will out-of-gas revert if recipient is a contract with logic inside receive()
- 367:            payable(destination).transfer(amount);
- 368:        } else {
- 369:            IERC20(token).safeTransfer(destination, amount);
- 370:        }

+         // will out-of-gas revert if recipient is a contract with logic inside receive()
+      address(token) == ETH_CONSTANT
+            ? payable(destination).transfer(amount)
+            : IERC20(token).safeTransfer(destination, amount);
```

### Instance#2 :

```solidity
File: contracts/NFTBoostVault.sol

589:  if (change > 0) {
            votingPower.push(registration.delegatee, delegateeVotes + uint256(change));
591:        } else {
            // if the change is negative, we multiply by -1 to avoid underflow when casting
            votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));
594:        }

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L589C9-L594C10

Recommended Changes : Remove if-else and use conditional like above Instance#1 recommendation.

## [G-08] Use `delete` for state variables to reinitialize with their default value rather than assigning default value saves gas.

This gas-saving behavior is due to how the delete operation is implemented in the Ethereum Virtual Machine (EVM).

When you use the delete keyword on a state variable, the EVM internally performs a low-level operation called SSTORE to
update the variable's storage slot. The SSTORE operation clears the storage slot, effectively resetting the variable to
its default value.

In contrast, if you manually assign a default value to a state variable, it involves writing a value to the storage
slot. This operation is performed using the SSTORE opcode as well but with an extra cost compared to the delete
operation. Writing the default value to the storage slot consumes more gas than simply clearing the storage slot with
the delete operation.

_12 instances - 2 files:_

### Instance#1-8:

```diff
File: contracts/ARCDVestingVault.sol

  177:         // delete the grant
- 178:         grant.allocation = 0;
- 179:         grant.cliffAmount = 0;
- 180:         grant.withdrawn = 0;
- 181:         grant.created = 0;
- 182:         grant.expiration = 0;
- 183:         grant.cliff = 0;
- 184:         grant.latestVotingPower = 0;
- 185:         grant.delegatee = address(0);

+ 178:        delete grant.allocation;
+ 179:        delete grant.cliffAmount;
+ 180:        delete grant.withdrawn;
+ 181:        delete grant.created;
+ 182:        delete grant.expiration;
+ 183:        delete grant.cliff;
+ 184:        delete grant.latestVotingPower;
+ 185:        delete grant.delegatee;

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L177C9-L185C38

### Instance#9-12:

```solidity
File:  contracts/NFTBoostVault.sol

251:          registration.amount = 0;
252:          registration.latestVotingPower = 0;
253:          registration.withdrawn = 0;
254:          registration.delegatee = address(0);
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L251C12-L254C49

Recommended Changes: Use `delete` instead of default value initialize to assign default value. Like above Instance#1-8
recommendation.

## [G-09] Use unchecked{} whenever underflow/overflow not possible

Because of previous condition check before the operation(+/-), it can be decided that underflow/overflow not possible.

_2 instances - 1 file:_

### Instance#1-2:`block.number - grant.cliff` and `grant.expiration - grant.cliff` can't underflow due to line 319 and 323 if() condition. So they can be marked unchecked to save gas.

If both line 319 323 if() conditions are false ie. `block.number` is greater than `grant.cliff ` and `grant.expiration`
is greater than `block.number` ie. `grant.expiration` also greater than `grant.cliff` so line 328 and 329 both
subtraction can't underflow so they can be marked unchecked safely to save gas.

```solidity
File: contracts/ARCDVestingVault.sol

319:      if (block.number < grant.cliff) {
            return 0;
        }
        // if after expiration, return the full allocation minus what has already been withdrawn
323:         if (block.number >= grant.expiration) {
            return grant.allocation - grant.withdrawn;
        }
        // if after cliff, return vested amount minus what has already been withdrawn
        uint256 postCliffAmount = grant.allocation - grant.cliffAmount;
328:        uint256 blocksElapsedSinceCliff = block.number - grant.cliff;
329:        uint256 totalBlocksPostCliff = grant.expiration - grant.cliff;

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L319C3-L329C71

Recommended Changes: Use unchecked block for line 328 and 329

```diff
+          unchecked{
328:               uint256 blocksElapsedSinceCliff = block.number - grant.cliff;
329:               uint256 totalBlocksPostCliff = grant.expiration - grant.cliff;
+              }

```

## [G-10] Use nested if and, avoid multiple check combinations using &&

Using nested if is cheaper gas wise than using && multiple check combinations. There are more advantages, such as easier
to read code and better coverage reports.

_5 instances - 1 file:_

### Instance#1:

```solidity
File:  contracts/NFTBoostVault.sol

247:  if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
248:                _withdrawNft();
249:            }

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L247C13-L249C14

Recommended Changes: Use nested if

```diff
- 247:       if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
+            if (registration.tokenAddress != address(0)) {
+                   if (registration.tokenId != 0) {
  248:                _withdrawNft();
  249:               }
+                }
```

### Instance#2:

```solidity
File: contracts/NFTBoostVault.sol

317:  if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
318:            // withdraw the current ERC1155 from the registration
319:            _withdrawNft();
        }
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L317C9-L320C10

Recommended Changes: Use nested if like above Instance#1.

### Instance#3:

```solidity
File: contracts/NFTBoostVault.sol

472: if (_tokenAddress != address(0) && _tokenId != 0) {
         if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();

            multiplier = getMultiplier(_tokenAddress, _tokenId);

            if (multiplier == 0) revert NBV_NoMultiplierSet();
478:        }
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L472

Recommended Changes: Use nested if like above Instance#1.

### Instance#4:

```solidity
File: contracts/NFTBoostVault.sol

632: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
633:        return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;
634:        }
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L632

Recommended Changes: Use nested if like above Instance#1.

### Instance#5:

```solidity
File: contracts/NFTBoostVault.sol

659: if (tokenAddress != address(0) && tokenId != 0) {
            _lockNft(from, tokenAddress, tokenId, nftAmount);
661:        }
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L659

Recommended Changes: Use nested if like above Instance#1.
