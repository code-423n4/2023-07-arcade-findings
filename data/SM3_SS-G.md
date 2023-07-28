
# Gas Optimizations Summary

| NO |  Issue  | Instances |
|----|---------|-----------|
|[G-01]| Multiple accesses of a mapping/array should use a local variable cache  | 4 | 
|[G-02]| Avoid emitting storage values (130 gas)  | 11 | 
|[G-03]| IF’s/require() statements that check input arguments should be at the top of the function  | 2 | 
|[G-04]| With assembly, .call (bool success) transfer can be done gas-optimized  | 1 | 
|[G-05]| A modifier used only once and not being inherited should be inlined to save gas  | 1 | 
|[G-06]| Using delete instead setting 0  | 13 | 
|[G-07]| Move owner checks to a modifier for gas efficant  | 1 | 
|[G-08]| x += y/x -= y costs more gas than x = x + y/x = x - y for state variables  | 17 | 
|[G-09]| Use assembly for loops  | 3 | 
|[G-10]| Use assembly to write address storage values  | 5 | 
|[G-11]| Inverting the condition of an if-else-statement wastes gas  | 3 | 
|[G-12]| Using > 0 costs more gas than != 0 when used on a uint in a require() statement  | 2 | 
|[G-13]| Use shift Right/Left instead of division/multiplication if possible  | 4 | 
|[G-14]| Use hardcode address instead address(this)  | 6 | 
|[G-15]| Use unchecked arithmitic when it is safe  | 17 | 
|[G-16]| Unnecessary look up in if condition  | 3 | 
|[G-17]| Amounts should be checked for 0 before calling a transfer  | 7 | 
|[G-18]| Access mappings directly rather than using accessor functions  | 1 | 
|[G-19]| Using XOR (^) and OR  bitwise equivalents  | 5 | 
|[G-20]| Using a positive conditional flow to save a NOT opcode  | 2 | 
|[G-21]| Using unchecked blocks to save gas  | 10 | 
|[G-22]| Combine events to save 2 Glogtopic (375 gas)  | 1 | 
|[G-23]| Cache state variables outside of loop to avoid reading/writing storage on every iteration  | 3 | 
|[G-24]| Use assembly to validate msg.sender  | 4 | 
|[G-25]| Use assembly to perform efficient back-to-back calls  | 7 | 
|[G-26]| Create immutable variable to avoid redundant external calls  | 1 | 
|[G-27]| State variables can be packed to use fewer storage slots  | 1 | 




## [G-01] Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping’s value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

To achieve this, declare a storage pointer for the variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling stakes[tokenId_], save its reference via a storage pointer: StakeInfo storage stakeInfo = stakes[tokenId_] and use the pointer instead.

### Cache amountClaimed mapping instead of access multiple time.
```solidity
file:  contracts/nft/ReputationBadge.sol

111             if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
            revert RB_InvalidClaimAmount(amount, totalClaimable);
        }

        // increment amount claimed
        amountClaimed[recipient][tokenId] += amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L111-L116

### Cache slastAllowanceSet,gscAllowance and blockExpenditure mapping instead of access multiple time.

```solidity
file:

        // enforce cool down period
308        if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {
            revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);
        }

        uint256 spendLimit = spendThresholds[token].small;
        // new limit cannot be more than the small threshold
        if (newAllowance > spendLimit) {
            revert T_InvalidAllowance(newAllowance, spendLimit);
        }

        // update allowance state
        lastAllowanceSet[token] = uint48(block.timestamp);
        gscAllowance[token] = newAllowance;

        emit GSCAllowanceUpdated(token, newAllowance);
    }

281            if (thresholds.small < gscAllowance[token]) {
            gscAllowance[token] = thresholds.small;

            emit GSCAllowanceUpdated(token, thresholds.small);
        }


360           uint256 spentThisBlock = blockExpenditure[block.number];
        if (amount + spentThisBlock > limit) revert T_BlockSpendLimit();
        blockExpenditure[block.number] = amount + spentThisBlock;
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L308-L323

## [G-02] Avoid emitting storage values (130 gas)

Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

### emit _newMinter instead of minter to save gas 

```solidity

file:  contracts/token/ArcadeToken.sol

136   emit MinterUpdated(minter);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136

### Cache grant.delegatee and emit cached value instead of reading from storage

```solidity
file:   contracts/ARCDVestingVault.sol

148    emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));

356    emit VoteChange(grant.delegatee, who, change);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L148

###  Cache registration.delegatee and emit cached value instead of reading from storage

```solidity
file:   contracts/NFTBoostVault.sol

195     emit VoteChange(msg.sender, registration.delegatee, -1 * int256(uint256(registration.latestVotingPower)));

598     emit VoteChange(who, registration.delegatee, change);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L195

### Cache arcadeToken and emit cached value instead of reading from storage

```solidity
file:  contracts/token/ArcadeTokenDistributor.sol

81     emit Distribute(address(arcadeToken), _treasury, treasuryAmount);

97     emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);

113    emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);

129    emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);

146    emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);

163    emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L81


## [G-03] IF’s/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Cheaper to check the function parameter before making an external function cal

```solidity
file:  contracts/NFTBoostVault.sol

650         function _lockTokens(
        address from,
        uint256 amount,
        address tokenAddress,
        uint128 tokenId,
        uint128 nftAmount
    ) internal {
        token.transferFrom(from, address(this), amount);

        if (tokenAddress != address(0) && tokenId != 0) {
            _lockNft(from, tokenAddress, tokenId, nftAmount);
        }
    }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L650-L662

### Cheaper to check first they first if condation after this check another  parameter.


```solidity
file:  contracts/nft/ReputationBadge.sol

98         function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
        uint256 mintPrice = mintPrices[tokenId] * amount;
        uint48 claimExpiration = claimExpirations[tokenId];

        if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));
        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
        if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
            revert RB_InvalidClaimAmount(amount, totalClaimable);
        }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L98-L113

## [G-04] With assembly, .call (bool success) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity
file:   contracts/ArcadeTreasury.sol

341    (bool success, ) = targets[i].call(calldatas[i]);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341

## [G-05] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file:  contracts/NFTBoostVault.sol

701    modifier onlyAirdrop()

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L701

## [G-06] Using delete instead setting 0 

```solidity
file:     contracts/ARCDVestingVault.sol

133       grant.withdrawn = 0;

178        grant.allocation = 0;

179        grant.cliffAmount = 0;

180        grant.withdrawn = 0;

181        grant.created = 0;

182        grant.expiration = 0;

183        grant.cliff = 0;

184        grant.latestVotingPower = 0;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L133

```solidity
file:   contracts/NFTBoostVault.sol
 
251     registration.amount = 0;

252     registration.latestVotingPower = 0;

253     registration.withdrawn = 0;

499     registration.withdrawn = 0;

566     registration.tokenId = 0;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#l251

## [G-07] Move owner checks to a modifier for gas efficant

It’s better to use a modifier for simple owner checks for an easier inspection of functions. This is also more gas efficient as it does not control with external call.

```solidity
file:  contracts/nft/ReputationBadge.sol

76    if (_owner == address(0)) revert RB_ZeroAddress("owner");

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L76


## [G-08] x += y/x -= y costs more gas than x = x + y/x = x - y for state variables

```solidity
file:  contracts/ARCDVestingVault.sol

166   grant.withdrawn += uint128(withdrawable);

171   grant.withdrawn += uint128(remaining);

200  unassigned.data += amount;

242  grant.withdrawn += uint128(withdrawable);

244   grant.withdrawn += uint128(amount);


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L166

```solidity
file:  contracts/NFTBoostVault.sol

161    balance.data += amount;

164     registration.amount += amount;

241    registration.withdrawn += amount;

278    balance.data += amount;

281    registration.amount += amount;

505     balance.data += _amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L161 

```solidity
file:   contracts/nft/ReputationBadge.sol

116     amountClaimed[recipient][tokenId] += amount;


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L116

```solidity
file:   contracts/ArcadeTreasury.sol
 
117    gscAllowance[token] -= amount;

198     gscAllowance[token] -= amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L117

```solidity
file:  contracts/ARCDVestingVault.sol

141     unassigned.data -= amount;

215     unassigned.data -= amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L141 

```solidity
file:    contracts/NFTBoostVault.sol

239      balance.data -= amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L239

## [G-09] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.

```solidity
file:   contracts/ArcadeTreasury.sol

339             for (uint256 i = 0; i < targets.length; ++i) {
            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
            (bool success, ) = targets[i].call(calldatas[i]);
            // revert if a single call fails
            if (!success) revert T_CallFailed();
        }
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L339-L344

```solidity
file:   contracts/NFTBoostVault.sol

345             for (uint256 i = 0; i < userAddresses.length; ++i) {
            NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[userAddresses[i]];
            _syncVotingPower(userAddresses[i], registration);
        }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345-L348

```solidity
file:   contracts/nft/ReputationBadge.sol

144             for (uint256 i = 0; i < _claimData.length; i++) {
            // expiration check
            if (_claimData[i].claimExpiration <= block.timestamp) {
                revert RB_InvalidExpiration(_claimData[i].claimRoot, _claimData[i].tokenId);
            }

            claimRoots[_claimData[i].tokenId] = _claimData[i].claimRoot;
            claimExpirations[_claimData[i].tokenId] = _claimData[i].claimExpiration;
            mintPrices[_claimData[i].tokenId] = _claimData[i].mintPrice;
        }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144-L153


## [G-10] Use assembly to write address storage values

```solidity
file:   contracts/nft/BadgeDescriptor.sol

34        constructor(string memory _baseURI) {
        // Empty baseURI is allowed
        baseURI = _baseURI;
    }

57      function setBaseURI(string memory newBaseURI) external onlyOwner {
        baseURI = newBaseURI;


```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L34-L37

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L57-L59

###  Recommendation Code

```solidity

  34:     constructor(
- 58:         baseURI = _baseURI;
+                  assembly {                      
+                      sstore(vault. baseURI,_baseURI )
+                  }  


```

```solidity
file:    contracts/token/ArcadeToken.sol

113         constructor(address _minter, address _initialDistribution) ERC20("Arcade", "ARCD") ERC20Permit("Arcade") {
        if (_minter == address(0)) revert AT_ZeroAddress("minter");
        if (_initialDistribution == address(0)) revert AT_ZeroAddress("initialDistribution");

        minter = _minter;

132   function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

        minter = _newMinter;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L113-L117

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L132-L135


###  Recommendation Code

```solidity

  113:     constructor(
- 135:         minter = _minter;
+                  assembly {                      
+                      sstore(vault. minter,_minter )
+                  }  

```

### use assembly for arcadeToken state varaible to save gas 

```solidity
file:    contracts/token/ArcadeTokenDistributor.sol

174         function setToken(IArcadeToken _arcadeToken) external onlyOwner {
        if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");
        if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();

        arcadeToken = _arcadeToken;
    }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L174-L179

## [G‑11] Inverting the condition of an if-else-statement wastes gas

```solidity
file:   contracts/nft/BadgeDescriptor.sol

49     return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49

```solidity
file:   contracts/ARCDVestingVault.sol

125    delegatee = delegatee == address(0) ? who : delegatee;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L125

```solidity
file:   contracts/NFTBoostVault.sol

491     _delegatee = _delegatee == address(0) ? user : _delegatee;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L491

## [G‑12] Using > 0 costs more gas than != 0 when used on a uint in a require() statement

This change saves 6 gas per instance. The optimization works until solidity version 0.8.13 where there is a regression in gas costs.

```solidity
file:   contracts/NFTBoostVault.sol

589      if (change > 0) 

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L589

```solidity
file:    contracts/nft/BadgeDescriptor.sol

49        return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49

## [G-13] Use shift Right/Left instead of division/multiplication if possible

```solidity
file:  contracts/ARCDVestingVault.sol

330    uint256 unlocked = grant.cliffAmount + (postCliffAmount * blocksElapsedSinceCliff) / totalBlocksPostCliff;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L330

```solidity
file:   contracts/token/ArcadeToken.sol

154     uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L154

```solidity
file:   contracts/NFTBoostVault.sol

494    uint128 newVotingPower = (_amount * multiplier) / MULTIPLIER_DENOMINATOR;

633    return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L494 


## [G-14] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

https://twitter.com/transmissions11/status/1518507047943245824



```solidity
file:   contracts/ARCDVestingVault.sol

201    token.transferFrom(msg.sender, address(this), amount);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L201

```solidity
file:  contracts/nft/ReputationBadge.sol

167    uint256 balance = address(this).balance;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L167

```solidity
file:  contracts/token/ArcadeAirdrop.sol

66     uint256 unclaimed = token.balanceOf(address(this));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L66

```solidity
file:   contracts/NFTBoostVault.sol

557      address(this),

657       token.transferFrom(from, address(this), amount);

674       IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L557

## [G-15] Use unchecked arithmitic when it is safe

For example, those dealing with eth value will basically never overflow:

```solidity
file:  contracts/ARCDVestingVault.sol

166   grant.withdrawn += uint128(withdrawable);

171   grant.withdrawn += uint128(remaining);

200  unassigned.data += amount;

242  grant.withdrawn += uint128(withdrawable);

244   grant.withdrawn += uint128(amount);


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L166

```solidity
file:  contracts/NFTBoostVault.sol

161    balance.data += amount;

164     registration.amount += amount;

241    registration.withdrawn += amount;

278    balance.data += amount;

281    registration.amount += amount;

505     balance.data += _amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L161 

```solidity
file:   contracts/nft/ReputationBadge.sol

116     amountClaimed[recipient][tokenId] += amount;


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L116

```solidity
file:   contracts/ArcadeTreasury.sol
 
117    gscAllowance[token] -= amount;

198     gscAllowance[token] -= amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L117

```solidity
file:  contracts/ARCDVestingVault.sol

141     unassigned.data -= amount;

215     unassigned.data -= amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L141 

```solidity
file:    contracts/NFTBoostVault.sol

239      balance.data -= amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L239

## [G-16] Unnecessary look up in if condition

If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity
file:   contracts/NFTBoostVault.sol

306    if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);

422    if (tokenAddress == address(0) || tokenId == 0)

552    if (registration.tokenAddress == address(0) || registration.tokenId == 0)

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L306

## [G-17] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here:

```solidity 
file:   contracts/ArcadeTreasury.sol

367      payable(destination).transfer(amount);

369     IERC20(token).safeTransfer(destination, amount);

372     emit TreasuryTransfer(token, destination, amount);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L367

```solidity
file:  contracts/ARCDVestingVault.sol

201    token.transferFrom(msg.sender, address(this), amount);

217    token.safeTransfer(recipient, amount);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L201 

```solidity
file:  contracts/NFTBoostVault.sol

674    IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L674

```solidity
file:  contracts/nft/ReputationBadge.sol

171     payable(recipient).transfer(balance);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L171 


## [G-18] Access mappings directly rather than using accessor functions

Saves having to do two JUMP instructions, along with stack setup
There are 6 instances of this issue:


```solidity
file:  contracts/ArcadeTreasury.sol

288    spendThresholds[token] = thresholds;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L288


## [G-19] Using XOR (^) and OR (|) bitwise equivalents

Estimated savings: 73 gas

```solidity
file:   contracts/ArcadeTreasury.sol

276     if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small)

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L276

```solidity
file:  contracts/ARCDVestingVault.sol

109    if (cliff >= expiration || cliff < startTime) revert AVV_InvalidSchedule();

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L109

```solidity
file:   contracts/NFTBoostVault.sol

306    if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);

422    if (tokenAddress == address(0) || tokenId == 0) 

552    if (registration.tokenAddress == address(0) || registration.tokenId == 0)

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L306

## [G-20] Using a positive conditional flow to save a NOT opcode

Estimated savings: 3 gas
```solidity
file:  contracts/ArcadeTreasury.sol

343    if (!success) revert T_CallFailed();

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L343 

```solidity
file:   contracts/nft/ReputationBadge.sol

110    if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L110

## [G-21] Using unchecked blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

```solidity
file:  contracts/ARCDVestingVault.sol

170    uint256 remaining = grant.allocation - grant.withdrawn;

268    votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);

324    return grant.allocation - grant.withdrawn;

327    uint256 postCliffAmount = grant.allocation - grant.cliffAmount;

328    uint256 blocksElapsedSinceCliff = block.number - grant.cliff;

329    uint256 totalBlocksPostCliff = grant.expiration - grant.cliff;

332    return unlocked - grant.withdrawn;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L170

```solidity
file:  contracts/token/ArcadeToken.sol

119   mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;

151   mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L119

```solidity
file:   contracts/BaseVotingVault.sol

101    return votingPower.findAndClear(user, blockNumber, block.number - staleBlockLag);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L101 

## [G-22] Combine events to save 2 Glogtopic (375 gas)

The events below are only emitted once in the handleRewards function. We can combine the events into one singular event to save two Glogtopic (375 gas) that would otherwise be paid for the additional two events.

```solidity
file:  contracts/ArcadeTreasury.sol

284                emit GSCAllowanceUpdated(token, thresholds.small);
        }

        // Overwrite the spend limits for specified token
        spendThresholds[token] = thresholds;

        emit SpendThresholdsUpdated(token, thresholds);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L284-L290

## [G-23] Cache state variables outside of loop to avoid reading/writing storage on every iteration

Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration. In addition, for some instances we are also able to increment the cached variable in the loop and update the storage variable outside the loop to save 1 SSTORE per loop iteration.

Total Instances: 3

### Cache spendThresholds[targets[i]].small outside of loop to save 1 SLOAD per iteration

```solidity
file:   contracts/ArcadeTreasury.sol

339            for (uint256 i = 0; i < targets.length; ++i) {
            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
            (bool success, ) = targets[i].call(calldatas[i]);
            // revert if a single call fails
            if (!success) revert T_CallFailed();
        }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L339-L344

### Cache registration outside of loop to save 1 SLOAD per iteration

```solidity
file:    contracts/NFTBoostVault.sol

345           for (uint256 i = 0; i < userAddresses.length; ++i) {
            NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[userAddresses[i]];
            _syncVotingPower(userAddresses[i], registration);
        }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345-L348

### Cache  claimRoots, claimExpirations and mintPrices outside of loop to save 1 SLOAD per iteration

```solidity
file:   contracts/nft/ReputationBadge.sol#

144           for (uint256 i = 0; i < _claimData.length; i++) {
            // expiration check
            if (_claimData[i].claimExpiration <= block.timestamp) {
                revert RB_InvalidExpiration(_claimData[i].claimRoot, _claimData[i].tokenId);
            }

            claimRoots[_claimData[i].tokenId] = _claimData[i].claimRoot;
            claimExpirations[_claimData[i].tokenId] = _claimData[i].claimExpiration;
            mintPrices[_claimData[i].tokenId] = _claimData[i].mintPrice;
        }

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144-L153

## [G-24] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file:   contracts/NFTBoostVault.sol

702     if (msg.sender != Storage.addressPtr("airdrop").data) revert NBV_NotAirdrop();

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L702

```solidity
file:   contracts/token/ArcadeToken.sol

168    if (msg.sender != minter) revert AT_MinterNotCaller(minter);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L168

```solidity

file:  contracts/BaseVotingVault.sol

188    if (msg.sender != manager()) revert BVV_NotManager();

197    if (msg.sender != timelock()) revert BVV_NotTimelock();

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L188

## [G-25] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case we are also able to efficiently store both function signatures together in memory as one word, saving one MLOAD in the process.

Note: In order to do this optimization safely we will cache and restore the free memory pointer after we are done with our function calls.

```solidity
file:    contracts/NFTBoostVault.sol

208             registration.latestVotingPower = uint128(addedVotingPower);
        registration.delegatee = to;

323            registration.tokenAddress = newTokenAddress;
        registration.tokenId = newTokenId;

565            registration.tokenAddress = address(0);
        registration.tokenId = 0;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L208-L209

```solidity
file:

91             Storage.set(Storage.uint256Ptr("initialized"), 1);
        Storage.set(Storage.addressPtr("timelock"), timelock);
        Storage.set(Storage.addressPtr("manager"), manager);
        Storage.set(Storage.uint256Ptr("entered"), 1);
        Storage.set(Storage.uint256Ptr("locked"), 1);



251                registration.amount = 0;
            registration.latestVotingPower = 0;
            registration.withdrawn = 0;
            registration.delegatee = address(0);

```


```solidity
file:   contracts/ARCDVestingVault.sol

131             grant.allocation = amount;
        grant.cliffAmount = cliffAmount;
        grant.withdrawn = 0;
        grant.created = startTime;
        grant.expiration = expiration;
        grant.cliff = cliff;
        grant.latestVotingPower = newVotingPower;
        grant.delegatee = delegatee;

178             grant.allocation = 0;
        grant.cliffAmount = 0;
        grant.withdrawn = 0;
        grant.created = 0;
        grant.expiration = 0;
        grant.cliff = 0;
        grant.latestVotingPower = 0;
        grant.delegatee = address(0);        


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L131-L-138

## [G-26] Create immutable variable to avoid redundant external calls

In the instances below, an external call to retrieve the directory is performed each time _swap and _mint is invoked. We can avoid performing this call on each invocation by executing this external call once in the constructor and storing the directory as an immutable variable. Doing so will save 2 external calls each time didPay is called (didPay invokes _swap & _mint).

```solidity
file:  contracts/nft/ReputationBadge.sol

41   IBadgeDescriptor public descriptor;


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L41

## [G-27] State variables can be packed to use fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

Note: This optimization will save ~20_000 deployment gas. only runtime gas is bencharked since runtime gas is paid multiple times over, while deployment gas is only paid once.

```solidity
file:   contracts/token/ArcadeTokenDistributor.sol

32        uint256 public constant treasuryAmount = 25_500_000 ether;
    /// @notice A flag to indicate if the treasury has already been transferred to
    bool public treasurySent;

    /// @notice 0.6% of initial distribution is for the token development partner
    uint256 public constant devPartnerAmount = 600_000 ether;
    /// @notice A flag to indicate if the token development partner has already been transferred to.
    bool public devPartnerSent;

    /// @notice 15% of initial distribution is for the community rewards pool
    uint256 public constant communityRewardsAmount = 15_000_000 ether;
    /// @notice A flag to indicate if the community rewards pool has already been transferred to
    bool public communityRewardsSent;

    /// @notice 10% of initial distribution is for the community airdrop contract
    uint256 public constant communityAirdropAmount = 10_000_000 ether;
    /// @notice A flag to indicate if the community airdrop contract has already been transferred to
    bool public communityAirdropSent;

    /// @notice 16.2% of initial distribution is for the Arcade team
    uint256 public constant vestingTeamAmount = 16_200_000 ether;
    /// @notice A flag to indicate if the launch partners have already been transferred to
    bool public vestingTeamSent;

    /// @notice 32.7% of initial distribution is for Arcade's launch partners
    uint256 public constant vestingPartnerAmount = 32_700_000 ether;
    /// @notice A flag to indicate if the Arcade team has already been transferred to
    bool public vestingPartnerSent;

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L32-L59
