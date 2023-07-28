# Report


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 6 |
| [NC-2](#NC-2) | Constants in comparisons should appear on the left side | 29 |
| [NC-3](#NC-3) | delete keyword can be used instead of setting to 0 | 13 |
| [NC-4](#NC-4) | Return values of `approve()` not checked | 4 |
| [NC-5](#NC-5) | Variable names don't follow the Solidity style guide | 6 |
### <a name="NC-1"></a>[NC-1] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant
constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

*Instances (6)*:
```solidity
File: contracts/ArcadeTreasury.sol

48:     bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");

49:     bytes32 public constant GSC_CORE_VOTING_ROLE = keccak256("GSC_CORE_VOTING");

50:     bytes32 public constant CORE_VOTING_ROLE = keccak256("CORE_VOTING");

```

```solidity
File: contracts/nft/ReputationBadge.sol

44:     bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");

45:     bytes32 public constant BADGE_MANAGER_ROLE = keccak256("BADGE_MANAGER");

46:     bytes32 public constant RESOURCE_MANAGER_ROLE = keccak256("RESOURCE_MANAGER");

```

### <a name="NC-2"></a>[NC-2] Constants in comparisons should appear on the left side
Constants should appear on the left side of comparisons, to avoid accidental assignment

*Instances (29)*:
```solidity
File: contracts/ARCDVestingVault.sol

102:         if (amount == 0) revert AVV_InvalidAmount();

105:         if (startTime == 0) {

162:         if (grant.allocation == 0) revert AVV_NoGrantSet();

229:         if (amount == 0) revert AVV_InvalidAmount();

233:         if (grant.allocation == 0) revert AVV_NoGrantSet();

```

```solidity
File: contracts/ArcadeTreasury.sol

114:         if (amount == 0) revert T_ZeroAmount();

136:         if (amount == 0) revert T_ZeroAmount();

155:         if (amount == 0) revert T_ZeroAmount();

174:         if (amount == 0) revert T_ZeroAmount();

195:         if (amount == 0) revert T_ZeroAmount();

217:         if (amount == 0) revert T_ZeroAmount();

236:         if (amount == 0) revert T_ZeroAmount();

255:         if (amount == 0) revert T_ZeroAmount();

273:         if (thresholds.small == 0) revert T_ZeroAmount();

305:         if (newAllowance == 0) revert T_ZeroAmount();

```

```solidity
File: contracts/NFTBoostVault.sol

120:         if (amount == 0) revert NBV_ZeroAmount();

144:         if (amount == 0) revert NBV_ZeroAmount();

224:         if (getIsLocked() == 1) revert NBV_Locked();

225:         if (amount == 0) revert NBV_ZeroAmount();

267:         if (amount == 0) revert NBV_ZeroAmount();

306:         if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);

308:         if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();

422:         if (tokenAddress == address(0) || tokenId == 0) {

473:             if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();

477:             if (multiplier == 0) revert NBV_NoMultiplierSet();

552:         if (registration.tokenAddress == address(0) || registration.tokenId == 0)

588:         if (change == 0) return;

```

```solidity
File: contracts/nft/ReputationBadge.sol

141:         if (_claimData.length == 0) revert RB_NoClaimData();

```

```solidity
File: contracts/token/ArcadeToken.sol

148:         if (_amount == 0) revert AT_ZeroMintAmount();

```

### <a name="NC-3"></a>[NC-3] delete keyword can be used instead of setting to 0
It's clearer and reflects the intention of the programmer

*Instances (13)*:
```solidity
File: contracts/ARCDVestingVault.sol

133:         grant.withdrawn = 0;

178:         grant.allocation = 0;

179:         grant.cliffAmount = 0;

180:         grant.withdrawn = 0;

181:         grant.created = 0;

182:         grant.expiration = 0;

183:         grant.cliff = 0;

184:         grant.latestVotingPower = 0;

```

```solidity
File: contracts/NFTBoostVault.sol

251:             registration.amount = 0;

252:             registration.latestVotingPower = 0;

253:             registration.withdrawn = 0;

499:         registration.withdrawn = 0;

566:         registration.tokenId = 0;

```

### <a name="NC-4"></a>[NC-4] Return values of `approve()` not checked
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Instances (4)*:
```solidity
File: contracts/ArcadeTreasury.sol

200:         _approve(token, spender, amount, spendThresholds[token].small);

219:         _approve(token, spender, amount, spendThresholds[token].small);

238:         _approve(token, spender, amount, spendThresholds[token].medium);

257:         _approve(token, spender, amount, spendThresholds[token].large);

```

### <a name="NC-5"></a>[NC-5] Variable names don't follow the Solidity style guide
Constant variable should be all caps  each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)

*Instances (6)*:
```solidity
File: contracts/token/ArcadeTokenDistributor.sol

32:     uint256 public constant treasuryAmount = 25_500_000 ether;

37:     uint256 public constant devPartnerAmount = 600_000 ether;

42:     uint256 public constant communityRewardsAmount = 15_000_000 ether;

47:     uint256 public constant communityAirdropAmount = 10_000_000 ether;

52:     uint256 public constant vestingTeamAmount = 16_200_000 ether;

57:     uint256 public constant vestingPartnerAmount = 32_700_000 ether;

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) |  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()` | 1 |
| [L-2](#L-2) | Do not use deprecated library functions | 2 |
| [L-3](#L-3) | Empty Function Body - Consider commenting why | 4 |
| [L-4](#L-4) | _safeMint() should be used rather than _mint() | 3 |
| [L-5](#L-5) | Unsafe ERC20 operation(s) | 5 |
### <a name="L-1"></a>[L-1]  `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

*Instances (1)*:
```solidity
File: contracts/nft/ReputationBadge.sol

211:         bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));

```

### <a name="L-2"></a>[L-2] Do not use deprecated library functions

*Instances (2)*:
```solidity
File: contracts/ArcadeTreasury.sol

90:         _setupRole(ADMIN_ROLE, _timelock);

```

```solidity
File: contracts/nft/ReputationBadge.sol

79:         _setupRole(ADMIN_ROLE, _owner);

```

### <a name="L-3"></a>[L-3] Empty Function Body - Consider commenting why

*Instances (4)*:
```solidity
File: contracts/ArcadeGSCCoreVoting.sol

32:     ) CoreVoting(timelock, baseQuorum, minProposalPower, gsc, votingVaults) {}

```

```solidity
File: contracts/ArcadeGSCVault.sol

39:     ) GSCVault(coreVoting, votingPowerBound, owner) {}

```

```solidity
File: contracts/ArcadeTreasury.sol

397:     receive() external payable {}

```

```solidity
File: contracts/ImmutableVestingVault.sol

33:     ) ARCDVestingVault(_token, _stale, manager_, timelock_) {}

```

### <a name="L-4"></a>[L-4] _safeMint() should be used rather than _mint()
_mint() is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both OpenZeppelin and solmate have versions of this function

*Instances (3)*:
```solidity
File: contracts/nft/ReputationBadge.sol

119:         _mint(recipient, tokenId, amount, "");

```

```solidity
File: contracts/token/ArcadeToken.sol

122:         _mint(_initialDistribution, INITIAL_MINT_AMOUNT);

159:         _mint(_to, _amount);

```

### <a name="L-5"></a>[L-5] Unsafe ERC20 operation(s)

*Instances (5)*:
```solidity
File: contracts/ARCDVestingVault.sol

201:         token.transferFrom(msg.sender, address(this), amount);

```

```solidity
File: contracts/ArcadeTreasury.sol

367:             payable(destination).transfer(amount);

391:         IERC20(token).approve(spender, amount);

```

```solidity
File: contracts/NFTBoostVault.sol

657:         token.transferFrom(from, address(this), amount);

```

```solidity
File: contracts/nft/ReputationBadge.sol

171:         payable(recipient).transfer(balance);

```


## Medium Issues


| |Issue|Instances|
|-|:-|:-:|
| [M-1](#M-1) | Centralization Risk for trusted owners | 29 |
### <a name="M-1"></a>[M-1] Centralization Risk for trusted owners

#### Impact:
Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

*Instances (29)*:
```solidity
File: contracts/ArcadeTreasury.sol

44: contract ArcadeTreasury is IArcadeTreasury, AccessControl, ReentrancyGuard {

112:     ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {

134:     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

153:     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

172:     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

193:     ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {

215:     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

234:     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

253:     ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

269:     function setThreshold(address token, SpendThreshold memory thresholds) external onlyRole(ADMIN_ROLE) {

303:     function setGSCAllowance(address token, uint256 newAllowance) external onlyRole(ADMIN_ROLE) {

336:     ) external onlyRole(ADMIN_ROLE) nonReentrant {

```

```solidity
File: contracts/nft/BadgeDescriptor.sol

17: contract BadgeDescriptor is IBadgeDescriptor, Ownable {

57:     function setBaseURI(string memory newBaseURI) external onlyOwner {

```

```solidity
File: contracts/nft/ReputationBadge.sol

39: contract ReputationBadge is ERC1155, AccessControl, ERC1155Burnable, IReputationBadge {

140:     function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {

163:     function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {

184:     function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {

219:     ) public view override(ERC1155, AccessControl, IERC165) returns (bool) {

```

```solidity
File: contracts/token/ArcadeAirdrop.sol

62:     function reclaim(address destination) external onlyOwner {

75:     function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {

```

```solidity
File: contracts/token/ArcadeTokenDistributor.sol

23: contract ArcadeTokenDistributor is Ownable {

73:     function toTreasury(address _treasury) external onlyOwner {

89:     function toDevPartner(address _devPartner) external onlyOwner {

105:     function toCommunityRewards(address _communityRewards) external onlyOwner {

121:     function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {

138:     function toTeamVesting(address _vestingTeam) external onlyOwner {

155:     function toPartnerVesting(address _vestingPartner) external onlyOwner {

174:     function setToken(IArcadeToken _arcadeToken) external onlyOwner {

```

