# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Using bools for storage incurs overhead | 6 |
| [GAS-2](#GAS-2) | Cache array length outside of loop | 3 |
| [GAS-3](#GAS-3) | Don't initialize variables with default value | 3 |
| [GAS-4](#GAS-4) | Functions guaranteed to revert when called by normal users can be marked `payable` | 25 |
| [GAS-5](#GAS-5) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 1 |
| [GAS-6](#GAS-6) | Using `private` rather than `public` for constants, saves gas | 19 |
| [GAS-7](#GAS-7) | Use != 0 instead of > 0 for unsigned integer comparison | 2 |
### <a name="GAS-1"></a>[GAS-1] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (6)*:
```solidity
File: contracts/token/ArcadeTokenDistributor.sol

34:     bool public treasurySent;

39:     bool public devPartnerSent;

44:     bool public communityRewardsSent;

49:     bool public communityAirdropSent;

54:     bool public vestingTeamSent;

59:     bool public vestingPartnerSent;

```

### <a name="GAS-2"></a>[GAS-2] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (3)*:
```solidity
File: contracts/ArcadeTreasury.sol

339:         for (uint256 i = 0; i < targets.length; ++i) {

```

```solidity
File: contracts/NFTBoostVault.sol

345:         for (uint256 i = 0; i < userAddresses.length; ++i) {

```

```solidity
File: contracts/nft/ReputationBadge.sol

144:         for (uint256 i = 0; i < _claimData.length; i++) {

```

### <a name="GAS-3"></a>[GAS-3] Don't initialize variables with default value

*Instances (3)*:
```solidity
File: contracts/ArcadeTreasury.sol

339:         for (uint256 i = 0; i < targets.length; ++i) {

```

```solidity
File: contracts/NFTBoostVault.sol

345:         for (uint256 i = 0; i < userAddresses.length; ++i) {

```

```solidity
File: contracts/nft/ReputationBadge.sol

144:         for (uint256 i = 0; i < _claimData.length; i++) {

```

### <a name="GAS-4"></a>[GAS-4] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (25)*:
```solidity
File: contracts/ARCDVestingVault.sol

157:     function revokeGrant(address who) external virtual onlyManager {

197:     function deposit(uint256 amount) external onlyManager {

211:     function withdraw(uint256 amount, address recipient) external override onlyManager {

```

```solidity
File: contracts/ArcadeTreasury.sol

269:     function setThreshold(address token, SpendThreshold memory thresholds) external onlyRole(ADMIN_ROLE) {

303:     function setGSCAllowance(address token, uint256 newAllowance) external onlyRole(ADMIN_ROLE) {

```

```solidity
File: contracts/BaseVotingVault.sol

68:     function setTimelock(address timelock_) external onlyTimelock {

80:     function setManager(address manager_) external onlyTimelock {

```

```solidity
File: contracts/NFTBoostVault.sol

363:     function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {

378:     function unlock() external override onlyTimelock {

392:     function setAirdropContract(address newAirdropContract) external override onlyManager {

```

```solidity
File: contracts/nft/BadgeDescriptor.sol

57:     function setBaseURI(string memory newBaseURI) external onlyOwner {

```

```solidity
File: contracts/nft/ReputationBadge.sol

140:     function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {

163:     function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {

184:     function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {

```

```solidity
File: contracts/token/ArcadeAirdrop.sol

62:     function reclaim(address destination) external onlyOwner {

75:     function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {

```

```solidity
File: contracts/token/ArcadeToken.sol

132:     function setMinter(address _newMinter) external onlyMinter {

145:     function mint(address _to, uint256 _amount) external onlyMinter {

```

```solidity
File: contracts/token/ArcadeTokenDistributor.sol

73:     function toTreasury(address _treasury) external onlyOwner {

89:     function toDevPartner(address _devPartner) external onlyOwner {

105:     function toCommunityRewards(address _communityRewards) external onlyOwner {

121:     function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {

138:     function toTeamVesting(address _vestingTeam) external onlyOwner {

155:     function toPartnerVesting(address _vestingPartner) external onlyOwner {

174:     function setToken(IArcadeToken _arcadeToken) external onlyOwner {

```

### <a name="GAS-5"></a>[GAS-5] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (1)*:
```solidity
File: contracts/nft/ReputationBadge.sol

144:         for (uint256 i = 0; i < _claimData.length; i++) {

```

### <a name="GAS-6"></a>[GAS-6] Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*Instances (19)*:
```solidity
File: contracts/ArcadeTreasury.sol

48:     bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");

49:     bytes32 public constant GSC_CORE_VOTING_ROLE = keccak256("GSC_CORE_VOTING");

50:     bytes32 public constant CORE_VOTING_ROLE = keccak256("CORE_VOTING");

56:     uint48 public constant SET_ALLOWANCE_COOL_DOWN = 7 days;

```

```solidity
File: contracts/NFTBoostVault.sol

66:     uint128 public constant MAX_MULTIPLIER = 1.5e3;

69:     uint128 public constant MULTIPLIER_DENOMINATOR = 1e3;

```

```solidity
File: contracts/nft/ReputationBadge.sol

44:     bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");

45:     bytes32 public constant BADGE_MANAGER_ROLE = keccak256("BADGE_MANAGER");

46:     bytes32 public constant RESOURCE_MANAGER_ROLE = keccak256("RESOURCE_MANAGER");

```

```solidity
File: contracts/token/ArcadeToken.sol

80:     uint48 public constant MIN_TIME_BETWEEN_MINTS = 365 days;

83:     uint256 public constant MINT_CAP = 2;

86:     uint256 public constant PERCENT_DENOMINATOR = 100;

89:     uint256 public constant INITIAL_MINT_AMOUNT = 100_000_000 ether;

```

```solidity
File: contracts/token/ArcadeTokenDistributor.sol

32:     uint256 public constant treasuryAmount = 25_500_000 ether;

37:     uint256 public constant devPartnerAmount = 600_000 ether;

42:     uint256 public constant communityRewardsAmount = 15_000_000 ether;

47:     uint256 public constant communityAirdropAmount = 10_000_000 ether;

52:     uint256 public constant vestingTeamAmount = 16_200_000 ether;

57:     uint256 public constant vestingPartnerAmount = 32_700_000 ether;

```

### <a name="GAS-7"></a>[GAS-7] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (2)*:
```solidity
File: contracts/NFTBoostVault.sol

589:         if (change > 0) {

```

```solidity
File: contracts/nft/BadgeDescriptor.sol

49:         return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

```

