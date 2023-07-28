## Gas optimization 

## Summary

No | Issue |Instances|
|-|:-|:-:|:-:|
|G-1|Use assembly to validate msg.sender|6
|G-2|Use do while loops instead of for loops|3
|G-3| Using storage instead of memory for structs/arrays saves gas|7
|G-4| ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops|3
|G-5|++i costs less gas than i++, especially when it’s used in for-loops (--i/i-- too)|1
|G-6|on’t initialize variables with default value|3
|G-7| Use != 0 instead of > 0 for unsigned integer comparison|2
|G-8|Use hardcode address instead address(this)|6
|G-9|Unnecessary look up in if condition|5
|G-10|Checking msg.sender to not be zero address is redundant|2
|G-11| Use function instead of modifiers (4 instances)|3
|G-12|+= costs more gas than = + for state variables|10
|G-13|Bytes constants are more efficient than string constants|4
|G-14|Use assembly for loops|3
|G-15|Do not calculate constants|3

### [G-01] Use assembly to validate msg.sender

```solidity

file: contracts/BaseVotingVault.sol

188   if (msg.sender != manager()) revert BVV_NotManager();

197    if (msg.sender != timelock()) revert BVV_NotTimelock();

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L188
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L197

```solidity

file: contracts/NFTBoostVault.sol

308    if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();

702      if (msg.sender != Storage.addressPtr("airdrop").data) revert NBV_NotAirdrop();

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L308
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L702

```solidity

file: contracts/token/ArcadeToken.sol

168   if (msg.sender != minter) revert AT_MinterNotCaller(minter);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L168

### [G-02] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity

fille: contracts/ArcadeTreasury.sol

339   for (uint256 i = 0; i < targets.length; ++i) 

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L339

```solidity

file: contracts/NFTBoostVault.sol

345    for (uint256 i = 0; i < userAddresses.length; ++i) 

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

```solidity

file: contracts/nft/ReputationBadge.sol#

144   for (uint256 i = 0; i < _claimData.length; i++)
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144

### [G‑03] Using storage instead of memory for structs/arrays saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity

file: contracts/ArcadeGSCCoreVoting.sol

31   address[] memory votingVaults                            
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeGSCCoreVoting.sol#L31

```solidity

file: blob/main/contracts/NFTBoostVault.sol

628  NFTBoostVaultStorage.Registration memory registration


697   function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4)
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L628
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L697

```solidiyu
file: /contracts/nft/ReputationBadge.sol

129            
    function uri(uint256 tokenId) public view override(ERC1155, IReputationBadge) returns (string memory) 

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L129

```solidity

file:contracts/nft/BadgeDescriptor.sol#L34

34   constructor(string memory _baseURI) 

48  function tokenURI(uint256 tokenId) external view override returns (string memory) 

57    function setBaseURI(string memory newBaseURI) external onlyOwner 
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L34
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L48
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L57

### [G‑04]  ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

```solidity

file: contracts/nft/ReputationBadge.sol

144  for (uint256 i = 0; i < _claimData.length; i++) 

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144

```solidity

file: contracts/ArcadeTreasury.sol

339  for (uint256 i = 0; i < targets.length; ++i)
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L339

```solidity

file: contracts/NFTBoostVault.sol

345   for (uint256 i = 0; i < userAddresses.length; ++i) 

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

### [G-05] ++i costs less gas than i++, especially when it’s used in for-loops (--i/i-- too)

```solidity

file: contracts/nft/ReputationBadge.sol

144  for (uint256 i = 0; i < _claimData.length; i++) 


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144

### [G-06] Don’t initialize variables with default value

```solidity

file: contracts/ArcadeTreasury.sol

339 for (uint256 i = 0; i < targets.length; ++i) 

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L339

```solidity

file: contracts/NFTBoostVault.sol

345    for (uint256 i = 0; i < userAddresses.length; ++i) 
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

```solidity

file: contracts/nft/ReputationBadge.sol

144    for (uint256 i = 0; i < _claimData.length; i++) 
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144

### [G-07] Use != 0 instead of > 0 for unsigned integer comparison

```solidity

file: contracts/NFTBoostVault.sol

589   if (change > 0) 
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L589

```solidity

file: contracts/nft/BadgeDescriptor.sol

49    return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L49

### [G-08] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address


```solidity

file: contracts/ARCDVestingVault.sol

201   token.transferFrom(msg.sender, address(this), amount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L201

```solidity

file: contracts/NFTBoostVault.sol

657       token.transferFrom(from, address(this), amount);

674  IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L657
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L674

```solidity

file: contracts/token/ArcadeAirdrop.sol

66    uint256 unclaimed = token.balanceOf(address(this));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L66

```solidity

file: contracts/nft/ReputationBadge.sol

167     uint256 balance = address(this).balance;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L167

### [G-09] Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: contracts/ArcadeTreasury.sol


276    if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small)
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L276

```solidity

file: contracts/ARCDVestingVault.sol


109      if (cliff >= expiration || cliff < startTime) revert AVV_InvalidSchedule();
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L109

```solidity

file: contracts/NFTBoostVault.sol

306     if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);

422   if (tokenAddress == address(0) || tokenId == 0)

552      if (registration.tokenAddress == address(0) || registration.tokenId == 0)
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L306
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L422
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L552

### [G-10] Checking msg.sender to not be zero address is redundant
There is an instance where msg.sender is checked not to be zero address. This check is redundant as no private key is known for this address, hence there can be no transactions coming from the zero address. The following diff removes this redundant check.

```solidity

file: contracts/NFTBoostVault.sol

171   _lockTokens(msg.sender, uint256(amount), address(0), 0, 0);

286       _lockTokens(msg.sender, amount, address(0), 0, 0);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L171
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L286

### [G-11] Use function instead of modifiers (4 instances)
Deployment. Gas Saved: 115 926

Minimum Method Call. Gas Saved: 162

Average Method Call. Gas Saved: -264

Maximum Method Call. Gas Saved: -481

Overall gas change: 734 (2.459%)

```solidity

file: contracts/BaseVotingVault.sol

187     modifier onlyManager() {
        if (msg.sender != manager()) revert BVV_NotManager();

        _;
    }

    /**
     * @notice Modifier to check that the caller is the timelock.
     */
    modifier onlyTimelock() {
        if (msg.sender != timelock()) revert BVV_NotTimelock();



```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L187_L196

```solidity

file: contracts/NFTBoostVault.sol

701  modifier onlyAirdrop() {
        if (msg.sender != Storage.addressPtr("airdrop").data) revert NBV_NotAirdrop();



```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L701

```solidity

file: contracts/token/ArcadeToken.sol

167    modifier onlyMinter() {
        if (msg.sender != minter) revert AT_MinterNotCaller(minter);
        _;
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L167

### [G-12] += costs more gas than = + for state variables
use = + or = - instead to save gas

```solidity

file: contracts/ARCDVestingVault.sol

166 grant.withdrawn += uint128(withdrawable);

171    grant.withdrawn += uint128(remaining);

200       unassigned.data += amount;

242   grant.withdrawn += uint128(withdrawable);

244   grant.withdrawn += uint128(amount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L166
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L171
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L200
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L242
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L244

```solidity

file: contracts/NFTBoostVault.sol

161   balance.data += amount;

            // update registration amount
            registration.amount += amount;

241     registration.withdrawn += amount;  
         
278   balance.data += amount;

        // update registration amount
        registration.amount += amount;

505        balance.data += _amount;        
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L161-L164
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L241
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L278-L281
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L505

```solidit

file: contracts/nft/ReputationBadge.sol

116    amountClaimed[recipient][tokenId] += amount;

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L116

### [G-13] Bytes constants are more efficient than string constants
If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

```solidity

file: contracts/nft/ReputationBadge.sol

129    function uri(uint256 tokenId) public view override(ERC1155, IReputationBadge) returns (string memory) 


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L129

```solidity

file:  contracts/nft/BadgeDescriptor.sol


18     using Strings for uint256;

    event SetBaseURI(address indexed caller, string baseURI);

    // ============================================ STATE ==============================================

24    string public baseURI;
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L18-L24

```solidity

file: contracts/nft/BadgeDescriptor.sol


34     constructor(string memory _baseURI) 

48   function tokenURI(uint256 tokenId) external view override returns (string memory) {
        return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
    }

    /**
     * @notice An owner-only function for setting the string value of the base URI.
     *
     * @param newBaseURI              The new value of the base URI.
     */
57    function setBaseURI(string memory newBaseURI) external onlyOwner 
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L34
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L48-L57

### [G-14] Use assembly for loops
In the following instances, assembly is used for more gas efficient loops

```solidity

file: contracts/ArcadeTreasury.sol

339   for (uint256 i = 0; i < targets.length; ++i) 

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L339

```solidity

file: contracts/NFTBoostVault.sol

345  for (uint256 i = 0; i < userAddresses.length; ++i) 
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

```solidity

file: contracts/nft/ReputationBadge.sol

144      for (uint256 i = 0; i < _claimData.length; i++) 
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L144

### [G-15] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity

file: contracts/ARCDVestingVault.sol

269  emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L269

```solidity

file:contracts/NFTBoostVault.sol
195    emit VoteChange(msg.sender, registration.delegatee, -1 * int256(uint256(registration.latestVotingPower)));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L195

```solidity

file:contracts/NFTBoostVault.sol

593   votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L593
