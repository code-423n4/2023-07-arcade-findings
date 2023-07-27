# gas optimization

 ## Note:  The last 5 types were not found completely by the bots {17,18,19,20,21}
  I found that it was missed by the bots

# summary

|      |   issue   |   instance    |
|------|-----------|---------------|
|[G-01]| Avoid contract existence checks by using low level calls|6|
|[G-02]| Use do while loops instead of for loops|2|
|[G-03]| Can Make The Variable Outside The Loop To Save Gas |1|
|[G-04]| Use != 0 instead of > 0 for unsigned integer comparison|2|
|[G-05]| Using storage instead of memory for structs/arrays saves gas|2|
|[G-06]| Using delete instead of setting info struct to 0 saves gas|13|
|[G-07]| Multiple accesses of a mapping/array should use a local variable cache|2|
|[G-08]| With assembly, .call (bool success) transfer can be done gas-optimized|1|
|[G-09]| State variables can be packed to use fewer storage slots|2|
|[G-10]| Amounts should be checked for 0 before calling a transfer|16|
|[G-11]| Should Use Unchecked Block where Over/Underflow is not Possible|10|
|[G-12]| use of the transfer function|2|
|[G-13]| Use hardcode address instead address(this)|3|
|[G-14]| >=/<= costs less gas than >/<|6|
|[G-15]| Using a positive conditional flow to save a NOT opcode|2|
|[G-16]| Avoid emitting storage values|7|
|[G-17]| Use calldata instead of memory for function arguments that do not get mutated|1|
|[G-18]| Internal functions only called once can be inlined to save gas|1|
|[G-19]| Functions guaranteed to revert when called by normal users can be marked payable|22|
|[G-20]| <array>.length should not be looked up in every loop of a for-loop|2|
|[G-21]| State variables should be cached in stack variables rather than re-reading them from storage|2|

## [G‑01] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File: contracts/ArcadeTreasury.sol
369        IERC20(token).safeTransfer(destination, amount);

391        IERC20(token).approve(spender, amount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L369



```solidity
File:  contracts/NFTBoostVault.sol
308        if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();

473        if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();

556        IERC1155(registration.tokenAddress).safeTransferFrom(

674        IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));    
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L308

## [G-02] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
File: contracts/NFTBoostVault.sol
345          for (uint256 i = 0; i < userAddresses.length; ++i) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

### Recommend Mig
```
function updateVotingPower(address[] calldata userAddresses) public override {
    if (userAddresses.length > 50) revert NBV_ArrayTooManyElements();

    uint256 i = 0;
    do {
        NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[userAddresses[i]];
        _syncVotingPower(userAddresses[i], registration);
        i++;
    } while (i < userAddresses.length);
}
```

```solidity
File: contracts/nft/ReputationBadge.sol
144        for (uint256 i = 0; i < _claimData.length; i++) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144

### Recommend Mig
```
function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
    if (_claimData.length == 0) revert RB_NoClaimData();
    if (_claimData.length > 50) revert RB_ArrayTooLarge();

    uint256 i = 0;
    do {
        // expiration check
        if (_claimData[i].claimExpiration <= block.timestamp) {
            revert RB_InvalidExpiration(_claimData[i].claimRoot, _claimData[i].tokenId);
        }

        claimRoots[_claimData[i].tokenId] = _claimData[i].claimRoot;
        claimExpirations[_claimData[i].tokenId] = _claimData[i].claimExpiration;
        mintPrices[_claimData[i].tokenId] = _claimData[i].mintPrice;
        i++;
    } while (i < _claimData.length);

    emit RootsPublished(_claimData);
}
```


## [G-03] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```
```solidity
File: contracts/ArcadeTreasury.sol
341   (bool success, ) = targets[i].call(calldatas[i]);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341


## [G-04]Use != 0 instead of > 0 for unsigned integer comparison
it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation,
 while the > 0 comparison requires an additional subtraction operation.
  As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

Here's an example of how you can use != 0 instead of > 0:

```
contract MyContract {
    uint256 public myUnsignedInteger;
    
    function myFunction() public view returns (bool) {
        // Use != 0 instead of > 0
        return myUnsignedInteger != 0;
    }
}
```

```solidity
File: contracts/NFTBoostVault.sol
589        if (change > 0) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L589

```solidity
File: contracts/nft/BadgeDescriptor.sol
49        return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49

## [G-05] Using storage instead of memory for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

### note: in `NFTBoostVaultStorage` contract `Registration` is a struct
```solidity
File: contracts/NFTBoostVault.sol
609        NFTBoostVaultStorage.Registration memory registration

628        NFTBoostVaultStorage.Registration memory registration
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L609

## [G-06] Using delete instead of setting info struct to 0 saves gas
[Reffrence](https://code4rena.com/reports/2023-01-biconomy#g-09-using-delete-instead-of-setting-info-struct-to-0-saves-gas)

### note: in `ARCDVestingVaultStorage` contract `Grant` is a struct

```solidity
File: contracts/ARCDVestingVault.sol
133     grant.withdrawn = 0;

178     grant.allocation = 0;
        grant.cliffAmount = 0;
        grant.withdrawn = 0;
        grant.created = 0;
        grant.expiration = 0;
        grant.cliff = 0;
        grant.latestVotingPower = 0;
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L178-L184


### note: in `NFTBoostVaultStorage` contract `Registration` is a struct
```solidity
File: contracts/NFTBoostVault.sol
251         registration.amount = 0;
            registration.latestVotingPower = 0;
            registration.withdrawn = 0;

499        registration.withdrawn = 0;

566        registration.tokenId = 0;            
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L251-L253

## [G‑07] Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

### mapping: `lastAllowanceSet[token]`
```solidity
File: contracts/ArcadeTreasury.sol
308        if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {

309        revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L308-L309

## [G-08] With assembly, .call (bool success) transfer can be done gas-optimized
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
-   (bool success,) = dest.call{value:amount}("");
 bool success; 
 assembly {  
            success := call(gas(), dest, amount, 0, 0)
     }  


```solidity
File: contracts/ArcadeTreasury.sol
341   (bool success, ) = targets[i].call(calldatas[i]);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341

## [G-09] State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

```solidity
File: contracts/token/ArcadeToken.sol
    uint48 public constant MIN_TIME_BETWEEN_MINTS = 365 days;

    /// @notice Cap on the percentage of totalSupply that can be minted at each mint
    uint256 public constant MINT_CAP = 2;

    /// @notice The denominator for the percentage calculations
    uint256 public constant PERCENT_DENOMINATOR = 100;

    /// @notice the initial token mint amount for distribution.
    uint256 public constant INITIAL_MINT_AMOUNT = 100_000_000 ether;

    // ======================== State =========================

    /// @notice Minter contract address responsible for minting future tokens
    address public minter;

    /// @notice The timestamp after which minting may occur
    uint256 public mintingAllowedAfter;
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L80-L97

```solidity
File: contracts/token/ArcadeTokenDistributor.sol
    uint256 public constant treasuryAmount = 25_500_000 ether;
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

## [G-10] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
File: contracts/ArcadeTreasury.sol
367       payable(destination).transfer(amount);

369       IERC20(token).safeTransfer(destination, amount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L367


```solidity
File: contracts/ARCDVestingVault.sol
167        token.safeTransfer(who, withdrawable);

172        token.safeTransfer(msg.sender, remaining);

201        token.transferFrom(msg.sender, address(this), amount);

217        token.safeTransfer(recipient, amount);

252        token.safeTransfer(msg.sender, withdrawable);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L167

```solidity
File: contracts/NFTBoostVault.sol
258        token.safeTransfer(msg.sender, amount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L258

```solidity
File: contracts/nft/ReputationBadge.sol
171         payable(recipient).transfer(balance);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L171

```solidity
File: contracts/token/ArcadeAirdrop.sol
67        token.safeTransfer(destination, unclaimed);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L67

```solidity
File:  contracts/token/ArcadeTokenDistributor.sol
79        arcadeToken.safeTransfer(_treasury, treasuryAmount);

95        arcadeToken.safeTransfer(_devPartner, devPartnerAmount);

111       arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);

127       arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);

144       arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);

161       arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L79

## [G-11] Should Use Unchecked Block where Over/Underflow is not Possible
Issue
Since Solidity 0.8.0, all arithmetic operations revert on overflow and underflow by default.
However in places where overflow and underflow is not possible, it is better to use unchecked block to reduce the gas usage.
[Reference]: (https://docs.soliditylang.org/en/v0.8.15/control-structures.html#checked-or-unchecked-arithmetic)


```solidity
File: contracts/ARCDVestingVault.sol
170        uint256 remaining = grant.allocation - grant.withdrawn;

268        votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);

324        return grant.allocation - grant.withdrawn;

327        uint256 postCliffAmount = grant.allocation - grant.cliffAmount;

328        uint256 blocksElapsedSinceCliff = block.number - grant.cliff;

329        uint256 totalBlocksPostCliff = grant.expiration - grant.cliff;

332        return unlocked - grant.withdrawn;

346        uint256 newVotingPower = grant.allocation - grant.withdrawn;

350        int256 change = int256(newVotingPower) - int256(grant.latestVotingPower);

352        votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L170

## [G-12] use of the transfer function
The problem is with the use of the transfer function. The transfer function has a gas limit of 2300, which means that if the recipient of the transfer is a contract with a fallback function, and that function consumes more than 2300 gas, the transfer will fail and revert.

This can result in a situation where the fees are not transferred to the intended recipient, and they remain in the contract, locked forever.

To prevent this issue, it's recommended to use the send or call function instead of transfer. These functions allow specifying a custom gas limit and a return value to check if the transfer was successful.

```solidity
File: contracts/nft/ReputationBadge.sol
171          payable(recipient).transfer(balance);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L171

```solidity
File: contracts/ArcadeTreasury.sol
367            payable(destination).transfer(amount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L367

## [G-13] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
File: contracts/NFTBoostVault.sol
557        address(this),

657        token.transferFrom(from, address(this), amount);

674        IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L557

## [G-14] >=/<= costs less gas than >/<
The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for >, which uses opcodes LT and ISZERO, but only requires GT for <=.
[Reffrence](https://gist.github.com/CloudEllie/05fda0fdecba2ed7d85f68c709e46c8d#gas-18--costs-less-gas-than-)


```solidity
File: contracts/ARCDVestingVault.sol
115         if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);

213         if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L115

```solidity
File: contracts/NFTBoostVault.sol
232       if (balance.data < amount) revert NBV_InsufficientBalance();

236       if (withdrawable < amount) revert NBV_InsufficientWithdrawableBalance(withdrawable);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L232


```solidity
File: contracts/nft/ReputationBadge.sol
109        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L109

```solidity
File: contracts/token/ArcadeToken.sol
146        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146

## [G-15] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas
[Reffrence](https://code4rena.com/reports/2023-01-opensea#g-03-using-a-positive-conditional-flow-to-save-a-not-opcode)


```solidity
File: contracts/ArcadeTreasury.sol
343            if (!success) revert T_CallFailed();
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L343

```solidity
File: contracts/nft/ReputationBadge.sol
110        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L110

## [G-16] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

```solidity
File:  contracts/token/ArcadeToken.sol
136        emit MinterUpdated(minter);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136

```solidity
File: contracts/token/ArcadeTokenDistributor.sol
81        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);

97        emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);

113       emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);

129       emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);

146       emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);

163       emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L81




## [G-17] Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity
File: contracts/ArcadeTreasury.sol
334   address[] memory targets,
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L334


## [G-18] Internal functions only called once can be inlined to save gas
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
File: contracts/nft/ReputationBadge.sol
204  function _verifyClaim(
        address recipient,
        uint256 tokenId,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) internal view returns (bool) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L204-L209

## [G-19] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```solidity
File: contracts/ArcadeTreasury.sol
112      ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {

134      ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

153      ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

172      ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

193      ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {

215      ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {   

234      ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

253      ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

269      function setThreshold(address token, SpendThreshold memory thresholds) external onlyRole(ADMIN_ROLE) {

303      function setGSCAllowance(address token, uint256 newAllowance) external onlyRole(ADMIN_ROLE) {

336      ) external onlyRole(ADMIN_ROLE) nonReentrant {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L112

```solidity
File: contracts/ARCDVestingVault.sol
99      ) external onlyManager {

197     function deposit(uint256 amount) external onlyManager {

211     function withdraw(uint256 amount, address recipient) external override onlyManager {

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L99

```solidity
File: contracts/NFTBoostVault.sol
143     ) external override onlyAirdrop nonReentrant {

363     function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {

378     function unlock() external override onlyTimelock {

392     function setAirdropContract(address newAirdropContract) external override onlyManager {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L143

```solidity
File: contracts/nft/ReputationBadge.sol
140     function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {

163     function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {

184     function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L140

```solidity
File: contracts/token/ArcadeToken.sol
145     function mint(address _to, uint256 _amount) external onlyMinter {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L145

## [G-20] <array>.length should not be looked up in every loop of a for-loop
Caching the array length outside a loop saves reading it on each iteration.

```solidity
File: contracts/NFTBoostVault.sol
345         for (uint256 i = 0; i < userAddresses.length; ++i) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

```solidity
File: contracts/nft/ReputationBadge.sol
144         for (uint256 i = 0; i < _claimData.length; i++) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144

## [G-21] State variables should be cached in stack variables rather than re-reading them from storage
Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.


### state var: `baseURI`
```solidity
File: contracts/nft/BadgeDescriptor.sol
49         return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49

### state var: `mintingAllowedAfter`
```solidity
File: contracts/token/ArcadeToken.sol
146         if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146

