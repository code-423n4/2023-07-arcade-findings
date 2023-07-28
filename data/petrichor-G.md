
|      | Issue   | Instance |
|------|---------|----------|  
|[G-01]|Use calldata instead of memory for function arguments that do not get mutated|1|
|[G-02]|Internal functions only called once can be inlined to save gas|1|
|[G-03]|Functions guaranteed to revert when called by normal users can be marked payable|21|
|[G-04]| <array>.length should not be looked up in every loop of a for-loop|2|
|[G-05]|State variables should be cached in stack variables rather than re-reading them from storage|1|
|[G-06]|Use != 0 instead of > 0 for unsigned integer comparison|2|
|[G-07]|Use do while loops instead of for loops|1|
|[G-08]|Avoid emitting storage values|7|
|[G-09]|Using a positive conditional flow to save a NOT opcode|2|
|[G-10]|Amounts should be checked for 0 before calling a transfer|14|
|[G-11]|Multiple accesses of a mapping/array should use a local variable cache|2|
|[G-12]|Using storage instead of memory for structs/arrays saves gas|2|



## [G-01] Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity
334   address[] memory targets,
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L334

### missed from bots

## [G-02] Internal functions only called once can be inlined to save gas
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
204  function _verifyClaim(
        address recipient,
        uint256 tokenId,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) internal view returns (bool) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L204-L209

### missed from bots

## [G-03] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```solidity
197     function deposit(uint256 amount) external onlyManager {

211     function withdraw(uint256 amount, address recipient) external override onlyManager {

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L197

```solidity
143     ) external override onlyAirdrop nonReentrant {

363     function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {

378     function unlock() external override onlyTimelock {

392     function setAirdropContract(address newAirdropContract) external override onlyManager {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L143

```solidity
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
140     function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {

163     function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {

184     function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L140

```solidity
145     function mint(address _to, uint256 _amount) external onlyMinter {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L145

### missed from bots

## [G-04] <array>.length should not be looked up in every loop of a for-loop
Caching the array length outside a loop saves reading it on each iteration.

```solidity
345     for (uint256 i = 0; i < userAddresses.length; ++i) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

```solidity
144      for (uint256 i = 0; i < _claimData.length; i++) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144

### missed from bots

## [G-05] State variables should be cached in stack variables rather than re-reading them from storage
Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

```solidity
49     return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49

```solidity
146   if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L146

### missed from bots




## [G-06]Use != 0 instead of > 0 for unsigned integer comparison


it's generally more gas-efficient to use != 0 instead of > 0 when
comparing unsigned integer types.

```solidity
589        if (change > 0) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L589

```solidity
49        return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49







## [G-07] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
345          for (uint256 i = 0; i < userAddresses.length; ++i) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

## [G-08] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

```solidity
136        emit MinterUpdated(minter);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136

```solidity
81        emit Distribute(address(arcadeToken), _treasury, treasuryAmount);

97        emit Distribute(address(arcadeToken), _devPartner, devPartnerAmount);

113       emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);

129       emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);

146       emit Distribute(address(arcadeToken), _vestingTeam, vestingTeamAmount);

163       emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L81

## [G-09] Using a positive conditional flow to save a NOT opcode
Estimated savings: 3 gas

```solidity
343            if (!success) revert T_CallFailed();
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L343

```solidity
110        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L110


## [G-10] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

```solidity
258        token.safeTransfer(msg.sender, amount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L258

```solidity
171         payable(recipient).transfer(balance);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L171

```solidity
67        token.safeTransfer(destination, unclaimed);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L67

```solidity
167        token.safeTransfer(who, withdrawable);

172        token.safeTransfer(msg.sender, remaining);

201        token.transferFrom(msg.sender, address(this), amount);

217        token.safeTransfer(recipient, amount);

252        token.safeTransfer(msg.sender, withdrawable);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L167

```solidity
79        arcadeToken.safeTransfer(_treasury, treasuryAmount);

95        arcadeToken.safeTransfer(_devPartner, devPartnerAmount);

111       arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);

127       arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);

144       arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);

161       arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L79

## [G‑11] Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

```solidity
308        if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {

309        revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L308-L309

## [G-12] Using storage instead of memory for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
609        NFTBoostVaultStorage.Registration memory registration

628        NFTBoostVaultStorage.Registration memory registration
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L609

