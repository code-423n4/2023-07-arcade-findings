# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 589 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (change > 0) {

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 144 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        for (uint256 i = 0; i < _claimData.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 3 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 345 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        for (uint256 i = 0; i < userAddresses.length; ++i) {


Found in line 339 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        for (uint256 i = 0; i < targets.length; ++i) {


Found in line 144 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        for (uint256 i = 0; i < _claimData.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 4 

## [GAS] Use Custom Errors
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 67 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        if (manager_ == address(0)) revert AVV_ZeroAddress("manager");


Found in line 68 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        if (timelock_ == address(0)) revert AVV_ZeroAddress("timelock");


Found in line 101 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        if (who == address(0)) revert AVV_ZeroAddress("who");


Found in line 53 at 2023-07-arcadexyz/contracts/BaseVotingVault.sol:

        if (address(_token) == address(0)) revert BVV_ZeroAddress("token");


Found in line 69 at 2023-07-arcadexyz/contracts/BaseVotingVault.sol:

        if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");


Found in line 81 at 2023-07-arcadexyz/contracts/BaseVotingVault.sol:

        if (manager_ == address(0)) revert BVV_ZeroAddress("manager");


Found in line 88 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (timelock == address(0)) revert NBV_ZeroAddress("timelock");


Found in line 89 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (manager == address(0)) revert NBV_ZeroAddress("manager");


Found in line 145 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (user == address(0)) revert NBV_ZeroAddress("user");


Found in line 183 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (to == address(0)) revert NBV_ZeroAddress("to");


Found in line 702 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (msg.sender != Storage.addressPtr("airdrop").data) revert NBV_NotAirdrop();


Found in line 88 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (_timelock == address(0)) revert T_ZeroAddress("timelock");


Found in line 113 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (destination == address(0)) revert T_ZeroAddress("destination");


Found in line 135 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (destination == address(0)) revert T_ZeroAddress("destination");


Found in line 154 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (destination == address(0)) revert T_ZeroAddress("destination");


Found in line 173 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (destination == address(0)) revert T_ZeroAddress("destination");


Found in line 194 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (spender == address(0)) revert T_ZeroAddress("spender");


Found in line 216 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (spender == address(0)) revert T_ZeroAddress("spender");


Found in line 235 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (spender == address(0)) revert T_ZeroAddress("spender");


Found in line 254 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (spender == address(0)) revert T_ZeroAddress("spender");


Found in line 271 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (token == address(0)) revert T_ZeroAddress("token");


Found in line 304 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (token == address(0)) revert T_ZeroAddress("token");


Found in line 49 at 2023-07-arcadexyz/contracts/token/ArcadeAirdrop.sol:

        if (_governance == address(0)) revert AA_ZeroAddress("governance");


Found in line 64 at 2023-07-arcadexyz/contracts/token/ArcadeAirdrop.sol:

        if (destination == address(0)) revert AA_ZeroAddress("destination");


Found in line 75 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_treasury == address(0)) revert AT_ZeroAddress("treasury");


Found in line 91 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");


Found in line 107 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");


Found in line 123 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");


Found in line 140 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");


Found in line 157 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");


Found in line 175 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");


Found in line 114 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        if (_minter == address(0)) revert AT_ZeroAddress("minter");


Found in line 115 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        if (_initialDistribution == address(0)) revert AT_ZeroAddress("initialDistribution");


Found in line 133 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");


Found in line 147 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        if (_to == address(0)) revert AT_ZeroAddress("to");


Found in line 76 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        if (_owner == address(0)) revert RB_ZeroAddress("owner");


Found in line 77 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        if (_descriptor == address(0)) revert RB_ZeroAddress("descriptor");


Found in line 164 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        if (recipient == address(0)) revert RB_ZeroAddress("recipient");


Found in line 185 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        if (_descriptor == address(0)) revert RB_ZeroAddress("descriptor");

------------------------------------------------------------------------ 

### Mitigation 

Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.










# VULN 5 

## [GAS] Use calldata instead of memory for function arguments that do not get mutated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 317 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

    function _getWithdrawableAmount(ARCDVestingVaultStorage.Grant memory grant) internal view returns (uint256) {


Found in line 697 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

    function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {


Found in line 269 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

    function setThreshold(address token, SpendThreshold memory thresholds) external onlyRole(ADMIN_ROLE) {


Found in line 57 at 2023-07-arcadexyz/contracts/nft/BadgeDescriptor.sol:

    function setBaseURI(string memory newBaseURI) external onlyOwner {

------------------------------------------------------------------------ 

### Mitigation 

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.










# VULN 6 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 345 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        for (uint256 i = 0; i < userAddresses.length; ++i) {


Found in line 339 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        for (uint256 i = 0; i < targets.length; ++i) {


Found in line 144 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        for (uint256 i = 0; i < _claimData.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 7 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 67 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        if (manager_ == address(0)) revert AVV_ZeroAddress("manager");


Found in line 68 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        if (timelock_ == address(0)) revert AVV_ZeroAddress("timelock");


Found in line 101 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        if (who == address(0)) revert AVV_ZeroAddress("who");


Found in line 125 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        delegatee = delegatee == address(0) ? who : delegatee;


Found in line 53 at 2023-07-arcadexyz/contracts/BaseVotingVault.sol:

        if (address(_token) == address(0)) revert BVV_ZeroAddress("token");


Found in line 69 at 2023-07-arcadexyz/contracts/BaseVotingVault.sol:

        if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");


Found in line 81 at 2023-07-arcadexyz/contracts/BaseVotingVault.sol:

        if (manager_ == address(0)) revert BVV_ZeroAddress("manager");


Found in line 88 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (timelock == address(0)) revert NBV_ZeroAddress("timelock");


Found in line 89 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (manager == address(0)) revert NBV_ZeroAddress("manager");


Found in line 145 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (user == address(0)) revert NBV_ZeroAddress("user");


Found in line 152 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (registration.delegatee == address(0)) {


Found in line 183 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (to == address(0)) revert NBV_ZeroAddress("to");


Found in line 273 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (registration.delegatee == address(0)) revert NBV_NoRegistration();


Found in line 306 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);


Found in line 314 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (registration.delegatee == address(0)) revert NBV_NoRegistration();


Found in line 422 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (tokenAddress == address(0) || tokenId == 0) {


Found in line 491 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        _delegatee = _delegatee == address(0) ? user : _delegatee;


Found in line 552 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        if (registration.tokenAddress == address(0) || registration.tokenId == 0)


Found in line 88 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (_timelock == address(0)) revert T_ZeroAddress("timelock");


Found in line 113 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (destination == address(0)) revert T_ZeroAddress("destination");


Found in line 135 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (destination == address(0)) revert T_ZeroAddress("destination");


Found in line 154 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (destination == address(0)) revert T_ZeroAddress("destination");


Found in line 173 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (destination == address(0)) revert T_ZeroAddress("destination");


Found in line 194 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (spender == address(0)) revert T_ZeroAddress("spender");


Found in line 216 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (spender == address(0)) revert T_ZeroAddress("spender");


Found in line 235 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (spender == address(0)) revert T_ZeroAddress("spender");


Found in line 254 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (spender == address(0)) revert T_ZeroAddress("spender");


Found in line 271 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (token == address(0)) revert T_ZeroAddress("token");


Found in line 304 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

        if (token == address(0)) revert T_ZeroAddress("token");


Found in line 49 at 2023-07-arcadexyz/contracts/token/ArcadeAirdrop.sol:

        if (_governance == address(0)) revert AA_ZeroAddress("governance");


Found in line 64 at 2023-07-arcadexyz/contracts/token/ArcadeAirdrop.sol:

        if (destination == address(0)) revert AA_ZeroAddress("destination");


Found in line 75 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_treasury == address(0)) revert AT_ZeroAddress("treasury");


Found in line 91 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");


Found in line 107 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");


Found in line 123 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");


Found in line 140 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");


Found in line 157 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");


Found in line 175 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

        if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");


Found in line 114 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        if (_minter == address(0)) revert AT_ZeroAddress("minter");


Found in line 115 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        if (_initialDistribution == address(0)) revert AT_ZeroAddress("initialDistribution");


Found in line 133 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");


Found in line 147 at 2023-07-arcadexyz/contracts/token/ArcadeToken.sol:

        if (_to == address(0)) revert AT_ZeroAddress("to");


Found in line 76 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        if (_owner == address(0)) revert RB_ZeroAddress("owner");


Found in line 77 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        if (_descriptor == address(0)) revert RB_ZeroAddress("descriptor");


Found in line 164 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        if (recipient == address(0)) revert RB_ZeroAddress("recipient");


Found in line 185 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        if (_descriptor == address(0)) revert RB_ZeroAddress("descriptor");

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 8 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 342 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

    function updateVotingPower(address[] calldata userAddresses) public override {


Found in line 363 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

    function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {


Found in line 405 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

    function getIsLocked() public view override returns (uint256) {


Found in line 418 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

    function getMultiplier(address tokenAddress, uint128 tokenId) public view override returns (uint128) {


Found in line 40 at 2023-07-arcadexyz/contracts/ImmutableVestingVault.sol:

    function revokeGrant(address) public pure override {


Found in line 129 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

    function uri(uint256 tokenId) public view override(ERC1155, IReputationBadge) returns (string memory) {

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 9 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 166 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        grant.withdrawn += uint128(withdrawable);


Found in line 171 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        grant.withdrawn += uint128(remaining);


Found in line 200 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        unassigned.data += amount;


Found in line 242 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

            grant.withdrawn += uint128(withdrawable);


Found in line 244 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

            grant.withdrawn += uint128(amount);


Found in line 161 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

            balance.data += amount;


Found in line 164 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

            registration.amount += amount;


Found in line 241 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        registration.withdrawn += amount;


Found in line 278 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        balance.data += amount;


Found in line 281 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        registration.amount += amount;


Found in line 505 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        balance.data += _amount;


Found in line 116 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        amountClaimed[recipient][tokenId] += amount;

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 10 

## [GAS] Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 370 and 371 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        // which can be persisted through upgrades, even if they change storage layout

        return (ARCDVestingVaultStorage.mappingAddressToGrantPtr("grants"));


Found in line 540 and 541 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        // This call returns a storage mapping with a unique non overwrite-able storage location.

        return NFTBoostVaultStorage.mappingAddressToRegistrationPtr("registrations");

------------------------------------------------------------------------ 

### Mitigation 

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.










# VULN 11 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 201 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

        token.transferFrom(msg.sender, address(this), amount);


Found in line 557 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

            address(this),


Found in line 657 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        token.transferFrom(from, address(this), amount);


Found in line 674 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));


Found in line 66 at 2023-07-arcadexyz/contracts/token/ArcadeAirdrop.sol:

        uint256 unclaimed = token.balanceOf(address(this));


Found in line 167 at 2023-07-arcadexyz/contracts/nft/ReputationBadge.sol:

        uint256 balance = address(this).balance;

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 12 

## [GAS] Functions guaranteed to revert when called by normal users can be marked payable
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 62 at 2023-07-arcadexyz/contracts/token/ArcadeAirdrop.sol:

    function reclaim(address destination) external onlyOwner {


Found in line 75 at 2023-07-arcadexyz/contracts/token/ArcadeAirdrop.sol:

    function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {


Found in line 73 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

    function toTreasury(address _treasury) external onlyOwner {


Found in line 89 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

    function toDevPartner(address _devPartner) external onlyOwner {


Found in line 105 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

    function toCommunityRewards(address _communityRewards) external onlyOwner {


Found in line 121 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

    function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {


Found in line 138 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

    function toTeamVesting(address _vestingTeam) external onlyOwner {


Found in line 155 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

    function toPartnerVesting(address _vestingPartner) external onlyOwner {


Found in line 174 at 2023-07-arcadexyz/contracts/token/ArcadeTokenDistributor.sol:

    function setToken(IArcadeToken _arcadeToken) external onlyOwner {


Found in line 57 at 2023-07-arcadexyz/contracts/nft/BadgeDescriptor.sol:

    function setBaseURI(string memory newBaseURI) external onlyOwner {

------------------------------------------------------------------------ 

### Mitigation 

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.










# VULN 13 

## [GAS] Using private rather than public for constants, saves gas
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 36 at 2023-07-arcadexyz/contracts/BaseVotingVault.sol:

    uint256 public immutable staleBlockLag;

------------------------------------------------------------------------ 

### Mitigation 

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.










# VULN 14 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 368 at 2023-07-arcadexyz/contracts/ARCDVestingVault.sol:

    function _grants() internal pure returns (mapping(address => ARCDVestingVaultStorage.Grant) storage) {


Found in line 539 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

    function _getRegistrations() internal pure returns (mapping(address => NFTBoostVaultStorage.Registration) storage) {


Found in line 685 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

        returns (mapping(address => mapping(uint128 => NFTBoostVaultStorage.AddressUintUint)) storage)


Found in line 49 at 2023-07-arcadexyz/contracts/nft/BadgeDescriptor.sol:

        return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 15 

## [GAS] With assembly, .call (bool success) transfer can be done gas-optimized
------------------------------------------------------------------------ 

### Proof of concept 

Found in 2023-07-arcadexyz/contracts/ArcadeTreasury.sol (function batchCalls():

    ) external onlyRole(ADMIN_ROLE) nonReentrant {

        if (targets.length != calldatas.length) revert T_ArrayLengthMismatch();

        // execute a package of low level calls

        for (uint256 i = 0; i < targets.length; ++i) {

            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);

            (bool success, ) = targets[i].call(calldatas[i]);

            // revert if a single call fails

            if (!success) revert T_CallFailed();

        }

    }




------------------------------------------------------------------------ 

### Mitigation 

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.










# VULN 16 

## [GAS] Empty blocks should be removed or emit something
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 39 at 2023-07-arcadexyz/contracts/ArcadeGSCVault.sol:

    ) GSCVault(coreVoting, votingPowerBound, owner) {}


Found in line 397 at 2023-07-arcadexyz/contracts/ArcadeTreasury.sol:

    receive() external payable {}


Found in line 32 at 2023-07-arcadexyz/contracts/ArcadeGSCCoreVoting.sol:

    ) CoreVoting(timelock, baseQuorum, minProposalPower, gsc, votingVaults) {}


Found in line 33 at 2023-07-arcadexyz/contracts/ImmutableVestingVault.sol:

    ) ARCDVestingVault(_token, _stale, manager_, timelock_) {}

------------------------------------------------------------------------ 

### Mitigation 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).










# VULN 17 

## [GAS] bytes constants are more eficient than string constans
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 24 at 2023-07-arcadexyz/contracts/nft/BadgeDescriptor.sol:

    string public baseURI;

------------------------------------------------------------------------ 

### Mitigation 

If the data can fit in 32 bytes, the bytes32 data type can be used instead of bytes or strings, as it is less robust in terms of robustness.










# VULN 18 

## [GAS] Use assembly to write address storage values
------------------------------------------------------------------------ 

### Proof of concept 

Found in lines 82 to 87 at 2023-07-arcadexyz/contracts/NFTBoostVault.sol:

    constructor(
        IERC20 token,
        uint256 staleBlockLag,
        address timelock,
        address manager
    ) BaseVotingVault(token, staleBlockLag) {


Found in lines 35 to 39 at 2023-07-arcadexyz/contracts/ArcadeGSCVault.sol:

    constructor(
        ICoreVoting coreVoting,
        uint256 votingPowerBound,
        address owner
    ) GSCVault(coreVoting, votingPowerBound, owner) {}


Found in lines 26 to 32 at 2023-07-arcadexyz/contracts/ArcadeGSCCoreVoting.sol:

    constructor(
        address timelock,
        uint256 baseQuorum,
        uint256 minProposalPower,
        address gsc,
        address[] memory votingVaults
    ) CoreVoting(timelock, baseQuorum, minProposalPower, gsc, votingVaults) {}


Found in lines 28 to 33 at 2023-07-arcadexyz/contracts/ImmutableVestingVault.sol:

    constructor(
        IERC20 _token,
        uint256 _stale,
        address manager_,
        address timelock_
    ) ARCDVestingVault(_token, _stale, manager_, timelock_) {}


Found in lines 42 to 48 at 2023-07-arcadexyz/contracts/token/ArcadeAirdrop.sol:

    constructor(
        address _governance,
        bytes32 _merkleRoot,
        IERC20 _token,
        uint256 _expiration,
        INFTBoostVault _votingVault
    ) ArcadeMerkleRewards(_merkleRoot, _token, _expiration, _votingVault) {

------------------------------------------------------------------------ 

### Mitigation 

Use assembly to write address storage values. Here are a few reasons: * Reduced opcode usage: When using assembly, you can directly manipulate storage values using lower-level instructions like sstore (storage store) instead of relying on higher-level Solidity storage assignments. These direct operations typically result in fewer opcode executions, reducing gas costs. * Avoiding unnecessary checks: Solidity storage assignments often involve additional checks and operations, such as enforcing security modifiers or triggering events. By using assembly, you can bypass these additional checks and perform the necessary storage operations directly, resulting in gas savings. * Optimized packing: Assembly provides greater flexibility in packing and unpacking data structures. By carefully arranging and manipulating the storage layout in assembly, you can achieve more efficient storage utilization and minimize wasted storage space. * Fine-grained control: Assembly allows for precise control over gas-consuming operations. You can optimize gas usage by selecting specific instructions and minimizing unnecessary operations or data copying.
