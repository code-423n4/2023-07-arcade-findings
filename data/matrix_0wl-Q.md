## Non Critical Issues

|       | Issue                                                                                    |
| ----- | :--------------------------------------------------------------------------------------- |
| NC-1  | ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS                                            |
| NC-2  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                                     |
| NC-3  | ADD TO BLACKLIST FUNCTION                                                                |
| NC-4  | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)                                     |
| NC-5  | GENERATE PERFECT CODE HEADERS EVERY TIME                                                 |
| NC-6  | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS                                          |
| NC-7  | FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES                                  |
| NC-8  | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT |
| NC-9  | MISSING EVENT FOR CRITICAL PARAMETER CHANGE                                              |
| NC-10 | NO SAME VALUE INPUT CONTROL                                                              |
| NC-11 | Omissions in events                                                                      |
| NC-12 | SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC                                       |
| NC-13 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS                  |
| NC-14 | LINES ARE TOO LONG                                                                       |
| NC-15 | Use OZ MerkleTree implementation instead of creating a new one                           |

### [NC-1] ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS

#### Description:

Add to indexed parameter for countable Events

#### **Proof Of Concept**

```solidity
File: contracts/ARCDVestingVault.sol

148:         emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));

269:         emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));

281:         emit VoteChange(to, msg.sender, int256(uint256(grant.latestVotingPower)));

356:         emit VoteChange(grant.delegatee, who, change);

```

```solidity
File: contracts/ArcadeTreasury.sol

284:             emit GSCAllowanceUpdated(token, thresholds.small);

```

```solidity
File: contracts/BaseVotingVault.sol

41:     event VoteChange(address indexed from, address indexed to, int256 amount);

```

```solidity
File: contracts/NFTBoostVault.sol

195:         emit VoteChange(msg.sender, registration.delegatee, -1 * int256(uint256(registration.latestVotingPower)));

211:         emit VoteChange(msg.sender, to, int256(addedVotingPower));

370:         emit MultiplierSet(tokenAddress, tokenId, multiplierValue);

382:         emit WithdrawalsUnlocked();

395:         emit AirdropContractUpdated(newAirdropContract);

509:         emit VoteChange(user, _delegatee, int256(uint256(newVotingPower)));

598:         emit VoteChange(who, registration.delegatee, change);

```

### [NC-2] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

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

392:     function setAirdropContract(address newAirdropContract) external override onlyManager {

```

```solidity
File: contracts/nft/BadgeDescriptor.sol

57:     function setBaseURI(string memory newBaseURI) external onlyOwner {

```

```solidity
File: contracts/nft/ReputationBadge.sol

184:     function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {

```

```solidity
File: contracts/token/ArcadeAirdrop.sol

75:     function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {

```

```solidity
File: contracts/token/ArcadeToken.sol

132:     function setMinter(address _newMinter) external onlyMinter {

```

```solidity
File: contracts/token/ArcadeTokenDistributor.sol

174:     function setToken(IArcadeToken _arcadeToken) external onlyOwner {

```

### [NC-3] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

### [NC-4] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

#### Description:

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).

#### **Proof Of Concept**

```solidity
File: contracts/nft/BadgeDescriptor.sol

49:         return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

```

```solidity
File: contracts/nft/ReputationBadge.sol

211:         bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));

```

### [NC-5] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-6] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS

#### Description:

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

#### **Proof Of Concept**

```solidity
File: contracts/ArcadeTreasury.sol

48:     bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");

```

```solidity
File: contracts/nft/ReputationBadge.sol

44:     bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");
```

### [NC-7] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

#### Description:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

#### **Proof Of Concept**

```solidity
File: contracts/ARCDVestingVault.sol

5: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

7: import "./external/council/libraries/History.sol";

8: import "./external/council/libraries/Storage.sol";

10: import "./libraries/ARCDVestingVaultStorage.sol";

11: import "./libraries/HashedStorageReentrancyBlock.sol";

13: import "./interfaces/IARCDVestingVault.sol";

14: import "./BaseVotingVault.sol";

```

```solidity
File: contracts/ArcadeGSCCoreVoting.sol

5: import "./external/council/CoreVoting.sol";

```

```solidity
File: contracts/ArcadeGSCVault.sol

5: import "./external/council/vaults/GSCVault.sol";

```

```solidity
File: contracts/ArcadeTreasury.sol

5: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

6: import "@openzeppelin/contracts/access/AccessControl.sol";

7: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

9: import "./interfaces/IArcadeTreasury.sol";

```

```solidity
File: contracts/BaseVotingVault.sol

5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

7: import "./external/council/libraries/History.sol";

8: import "./external/council/libraries/Storage.sol";

10: import "./libraries/HashedStorageReentrancyBlock.sol";

12: import "./interfaces/IBaseVotingVault.sol";

```

```solidity
File: contracts/ImmutableVestingVault.sol

5: import "./ARCDVestingVault.sol";

```

```solidity
File: contracts/NFTBoostVault.sol

5: import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";

6: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

8: import "./external/council/libraries/History.sol";

9: import "./external/council/libraries/Storage.sol";

11: import "./libraries/NFTBoostVaultStorage.sol";

12: import "./interfaces/INFTBoostVault.sol";

13: import "./BaseVotingVault.sol";

```

```solidity
File: contracts/nft/BadgeDescriptor.sol

5: import "@openzeppelin/contracts/access/Ownable.sol";

6: import "@openzeppelin/contracts/utils/Strings.sol";

8: import "../interfaces/IBadgeDescriptor.sol";

```

```solidity
File: contracts/nft/ReputationBadge.sol

5: import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";

6: import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Burnable.sol";

7: import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

8: import "@openzeppelin/contracts/access/AccessControl.sol";

9: import "@openzeppelin/contracts/utils/Strings.sol";

11: import "../interfaces/IReputationBadge.sol";

12: import "../interfaces/IBadgeDescriptor.sol";

```

```solidity
File: contracts/token/ArcadeAirdrop.sol

5: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

7: import "../external/council/libraries/Authorizable.sol";

9: import "../libraries/ArcadeMerkleRewards.sol";

```

```solidity
File: contracts/token/ArcadeToken.sol

5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

6: import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";

7: import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";

9: import "../interfaces/IArcadeToken.sol";

```

```solidity
File: contracts/token/ArcadeTokenDistributor.sol

5: import "@openzeppelin/contracts/access/Ownable.sol";

6: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

8: import "../interfaces/IArcadeToken.sol";

```

#### Recommended Mitigation Steps:

`import {contract1 , contract2} from "filename.sol";` OR Use specific imports syntax per solidity docs recommendation.

### [NC-8] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### Description:

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

#### **Proof Of Concept**

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

211:         bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));

```

### [NC-9] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

#### Description:

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### **Proof Of Concept**

```solidity
File: contracts/ARCDVestingVault.sol

148:         emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));

269:         emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));

281:         emit VoteChange(to, msg.sender, int256(uint256(grant.latestVotingPower)));

356:         emit VoteChange(grant.delegatee, who, change);

```

```solidity
File: contracts/NFTBoostVault.sol

195:         emit VoteChange(msg.sender, registration.delegatee, -1 * int256(uint256(registration.latestVotingPower)));

211:         emit VoteChange(msg.sender, to, int256(addedVotingPower));

370:         emit MultiplierSet(tokenAddress, tokenId, multiplierValue);

382:         emit WithdrawalsUnlocked();

395:         emit AirdropContractUpdated(newAirdropContract);

509:         emit VoteChange(user, _delegatee, int256(uint256(newVotingPower)));

598:         emit VoteChange(who, registration.delegatee, change);

```

### [NC-10] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: contracts/BaseVotingVault.sol

56:         token = _token;

57:         staleBlockLag = _staleBlockLag;

98:         History.HistoricalBalances memory votingPower = _votingPower();

114:         History.HistoricalBalances memory votingPower = _votingPower();

```

```solidity
File: contracts/NFTBoostVault.sol

159:             Storage.Uint256 storage balance = _balance();

190:         History.HistoricalBalances memory votingPower = _votingPower();

231:         Storage.Uint256 storage balance = _balance();

276:         Storage.Uint256 storage balance = _balance();

481:         Storage.Uint256 storage balance = _balance();

497:         registration.amount = _amount;

500:         registration.tokenId = _tokenId;

501:         registration.tokenAddress = _tokenAddress;

502:         registration.delegatee = _delegatee;

523:         History.HistoricalBalances memory votingPower = _votingPower();

580:         History.HistoricalBalances memory votingPower = _votingPower();

```

```solidity
File: contracts/nft/BadgeDescriptor.sol

36:         baseURI = _baseURI;

```

```solidity
File: contracts/token/ArcadeToken.sol

117:         minter = _minter;

```

```solidity
File: contracts/token/ArcadeTokenDistributor.sol

178:         arcadeToken = _arcadeToken;

```

### [NC-11] Omissions in events

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

The events should include the new value and old value where possible.

#### **Proof Of Concept**

```solidity
File: contracts/NFTBoostVault.sol

382:         emit WithdrawalsUnlocked();

395:         emit AirdropContractUpdated(newAirdropContract);

509:         emit VoteChange(user, _delegatee, int256(uint256(newVotingPower)));

```

```solidity
File: contracts/nft/ReputationBadge.sol

155:         emit RootsPublished(_claimData);

```

```solidity
File: contracts/token/ArcadeAirdrop.sol

78:         emit SetMerkleRoot(_merkleRoot);

```

```solidity
File: contracts/token/ArcadeToken.sol

136:         emit MinterUpdated(minter);

```

### [NC-12] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

#### Description:

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

#### Exploit Scenario:

A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### **Proof Of Concept**

```solidity
File: hardhat.config.ts

    solidity: {
        compilers: [
            {
                version: "0.8.18",
                settings: {
                    metadata: {
                        // Not including the metadata hash
                        // https://github.com/paulrberg/solidity-template/issues/31
                        bytecodeHash: "none",
                    },
                    // You should disable the optimizer when debugging
                    // https://hardhat.org/hardhat-network/#solidity-optimizer-support
                    optimizer: {
                        enabled: optimizerEnabled,
                        runs: 999999,
                    },
                    //viaIR: true, // experimental compiler feature to reduce stack 2 deep intolerance
                },
            },
        ],
    },
```

#### Recommended Mitigation Steps

Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug.

Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

### [NC-13] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: contracts/ArcadeTreasury.sol

53:     address internal constant ETH_CONSTANT = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

```

### [NC-14] LINES ARE TOO LONG

#### Description:

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

[Reference](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length)

#### **Proof Of Concept**

```solidity
File: contracts/ARCDVestingVault.sol

48: contract ARCDVestingVault is IARCDVestingVault, HashedStorageReentrancyBlock, BaseVotingVault {

66:     constructor(IERC20 _token, uint256 _stale, address manager_, address timelock_) BaseVotingVault(_token, _stale) {

109:         if (cliff >= expiration || cliff < startTime) revert AVV_InvalidSchedule();

115:         if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);

211:     function withdraw(uint256 amount, address recipient) external override onlyManager {

213:         if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);

234:         if (grant.cliff > block.number) revert AVV_CliffNotReached(grant.cliff);

238:         if (amount > withdrawable) revert AVV_InsufficientBalance(withdrawable);

268:         votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);

269:         emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));

281:         emit VoteChange(to, msg.sender, int256(uint256(grant.latestVotingPower)));

304:     function getGrant(address who) external view returns (ARCDVestingVaultStorage.Grant memory) {

317:     function _getWithdrawableAmount(ARCDVestingVaultStorage.Grant memory grant) internal view returns (uint256) {

330:         uint256 unlocked = grant.cliffAmount + (postCliffAmount * blocksElapsedSinceCliff) / totalBlocksPostCliff;

341:     function _syncVotingPower(address who, ARCDVestingVaultStorage.Grant storage grant) internal {

350:         int256 change = int256(newVotingPower) - int256(grant.latestVotingPower);

352:         votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));

368:     function _grants() internal pure returns (mapping(address => ARCDVestingVaultStorage.Grant) storage) {

```

```solidity
File: contracts/ArcadeTreasury.sol

49:     bytes32 public constant GSC_CORE_VOTING_ROLE = keccak256("GSC_CORE_VOTING");

53:     address internal constant ETH_CONSTANT = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

71:     event SpendThresholdsUpdated(address indexed token, SpendThreshold thresholds);

74:     event TreasuryTransfer(address indexed token, address indexed destination, uint256 amount);

77:     event TreasuryApproval(address indexed token, address indexed spender, uint256 amount);

269:     function setThreshold(address token, SpendThreshold memory thresholds) external onlyRole(ADMIN_ROLE) {

276:         if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small) {

303:     function setGSCAllowance(address token, uint256 newAllowance) external onlyRole(ADMIN_ROLE) {

308:         if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {

309:             revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);

340:             if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);

358:     function _spend(address token, uint256 amount, address destination, uint256 limit) internal {

384:     function _approve(address token, address spender, uint256 amount, uint256 limit) internal {

```

```solidity
File: contracts/BaseVotingVault.sol

14: import { BVV_NotManager, BVV_NotTimelock, BVV_ZeroAddress, BVV_UpperLimitBlock } from "./errors/Governance.sol";

24: abstract contract BaseVotingVault is HashedStorageReentrancyBlock, IBaseVotingVault {

54:         if (_staleBlockLag >= block.number) revert BVV_UpperLimitBlock(_staleBlockLag);

96:     function queryVotePower(address user, uint256 blockNumber, bytes calldata) external override returns (uint256) {

101:         return votingPower.findAndClear(user, blockNumber, block.number - staleBlockLag);

112:     function queryVotePowerView(address user, uint256 blockNumber) external view returns (uint256) {

179:     function _votingPower() internal pure returns (History.HistoricalBalances memory) {

```

```solidity
File: contracts/NFTBoostVault.sol

122:         _registerAndDelegate(msg.sender, amount, tokenId, tokenAddress, delegatee);

148:         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[user];

156:             if (delegatee != registration.delegatee) revert NBV_WrongDelegatee(delegatee, registration.delegatee);

185:         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

191:         uint256 oldDelegateeVotes = votingPower.loadTop(registration.delegatee);

194:         votingPower.push(registration.delegatee, oldDelegateeVotes - registration.latestVotingPower);

195:         emit VoteChange(msg.sender, registration.delegatee, -1 * int256(uint256(registration.latestVotingPower)));

228:         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

236:         if (withdrawable < amount) revert NBV_InsufficientWithdrawableBalance(withdrawable);

247:             if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

269:         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

305:     function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {

306:         if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);

308:         if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();

310:         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

317:         if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

342:     function updateVotingPower(address[] calldata userAddresses) public override {

346:             NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[userAddresses[i]];

363:     function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {

366:         NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];

392:     function setAirdropContract(address newAirdropContract) external override onlyManager {

418:     function getMultiplier(address tokenAddress, uint128 tokenId) public view override returns (uint128) {

419:         NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];

436:     function getRegistration(address who) external view override returns (NFTBoostVaultStorage.Registration memory) {

473:             if (IERC1155(_tokenAddress).balanceOf(user, _tokenId) == 0) revert NBV_DoesNotOwn();

484:         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[user];

494:         uint128 newVotingPower = (_amount * multiplier) / MULTIPLIER_DENOMINATOR;

521:     function _grantVotingPower(address delegatee, uint128 newVotingPower) internal {

539:     function _getRegistrations() internal pure returns (mapping(address => NFTBoostVaultStorage.Registration) storage) {

541:         return NFTBoostVaultStorage.mappingAddressToRegistrationPtr("registrations");

550:         NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

552:         if (registration.tokenAddress == address(0) || registration.tokenId == 0)

553:             revert NBV_InvalidNft(registration.tokenAddress, registration.tokenId);

579:     function _syncVotingPower(address who, NFTBoostVaultStorage.Registration storage registration) internal {

585:         int256 change = int256(newVotingPower) - int256(uint256(registration.latestVotingPower));

590:             votingPower.push(registration.delegatee, delegateeVotes + uint256(change));

593:             votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));

632:         if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

633:             return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;

673:     function _lockNft(address from, address tokenAddress, uint128 tokenId, uint128 nftAmount) internal {

674:         IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));

685:         returns (mapping(address => mapping(uint128 => NFTBoostVaultStorage.AddressUintUint)) storage)

688:         return NFTBoostVaultStorage.mappingAddressToPackedUintUint("multipliers");

697:     function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {

702:         if (msg.sender != Storage.addressPtr("airdrop").data) revert NBV_NotAirdrop();

```

```solidity
File: contracts/nft/BadgeDescriptor.sol

48:     function tokenURI(uint256 tokenId) external view override returns (string memory) {

49:         return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

```

```solidity
File: contracts/nft/ReputationBadge.sol

39: contract ReputationBadge is ERC1155, AccessControl, ERC1155Burnable, IReputationBadge {

46:     bytes32 public constant RESOURCE_MANAGER_ROLE = keccak256("RESOURCE_MANAGER");

108:         if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));

109:         if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);

110:         if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();

129:     function uri(uint256 tokenId) public view override(ERC1155, IReputationBadge) returns (string memory) {

140:     function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {

147:                 revert RB_InvalidExpiration(_claimData[i].claimRoot, _claimData[i].tokenId);

151:             claimExpirations[_claimData[i].tokenId] = _claimData[i].claimExpiration;

163:     function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {

184:     function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {

211:         bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));

```

```solidity
File: contracts/token/ArcadeToken.sol

113:     constructor(address _minter, address _initialDistribution) ERC20("Arcade", "ARCD") ERC20Permit("Arcade") {

115:         if (_initialDistribution == address(0)) revert AT_ZeroAddress("initialDistribution");

146:         if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);

154:         uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;

156:             revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);

```

```solidity
File: contracts/token/ArcadeTokenDistributor.sol

10: import { AT_AlreadySent, AT_ZeroAddress, AT_TokenAlreadySet } from "../errors/Token.sol";

107:         if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");

113:         emit Distribute(address(arcadeToken), _communityRewards, communityRewardsAmount);

123:         if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");

129:         emit Distribute(address(arcadeToken), _communityAirdrop, communityAirdropAmount);

157:         if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");

163:         emit Distribute(address(arcadeToken), _vestingPartner, vestingPartnerAmount);

175:         if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");

```

### [NC-15] Use OZ MerkleTree implementation instead of creating a new one

#### Description:

Instead of your own merkle tree lib use openzeppelin implementation;

https://docs.openzeppelin.com/contracts/4.x/api/utils#MerkleProof

#### **Proof Of Concept**

```solidity
File: contracts/token/ArcadeAirdrop.sol

9: import "../libraries/ArcadeMerkleRewards.sol";

21: contract ArcadeAirdrop is ArcadeMerkleRewards, Authorizable {

26:     event SetMerkleRoot(bytes32 indexed merkleRoot);

26:     event SetMerkleRoot(bytes32 indexed merkleRoot);

44:         bytes32 _merkleRoot,

48:     ) ArcadeMerkleRewards(_merkleRoot, _token, _expiration, _votingVault) {

48:     ) ArcadeMerkleRewards(_merkleRoot, _token, _expiration, _votingVault) {

75:     function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {

75:     function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {

76:         rewardsRoot = _merkleRoot;

78:         emit SetMerkleRoot(_merkleRoot);

78:         emit SetMerkleRoot(_merkleRoot);

```

## Low Issues

|      | Issue                                                                          |
| ---- | :----------------------------------------------------------------------------- |
| L-1  | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()                            |
| L-2  | USAGE OF PAYABLE.TRANSFER CAN LEAD TO LOSS OF FUNDS                            |
| L-3  | DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH       |
| L-4  | DRAFT OPENZEPPELIN DEPENDENCIES                                                |
| L-5  | INSUFFICIENT COVERAGE                                                          |
| L-6  | LOSS OF PRECISION DUE TO ROUNDING                                              |
| L-7  | OWNER CAN RENOUNCE OWNERSHIP                                                   |
| L-8  | THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS |
| L-9  | STACK TOO DEEP WHEN COMPILING                                                  |
| L-10 | Timestamp Dependence                                                           |
| L-11 | Account existence check for low-level calls                                    |
| L-12 | UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT                        |
| L-13 | USE `SAFETRANSFER` INSTEAD OF `TRANSFER`                                       |
| L-14 | NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES                 |

### [L-1] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()

#### Description:

Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

#### **Proof Of Concept**

```solidity
File: contracts/nft/ReputationBadge.sol

211:         bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));

```

### [L-2] USAGE OF PAYABLE.TRANSFER CAN LEAD TO LOSS OF FUNDS

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

```solidity
File: contracts/ArcadeTreasury.sol

367:             payable(destination).transfer(amount);

```

```solidity
File: contracts/nft/ReputationBadge.sol

171:         payable(recipient).transfer(balance);

```

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

### [L-3] DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH

#### Recommended Mitigation Steps:

Consider using a more generic implementation that can handle different hash functions and lengths and allow the user to choose.

```solidity
File: contracts/ArcadeTreasury.sol

53:     address internal constant ETH_CONSTANT = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

```

### [L-4] DRAFT OPENZEPPELIN DEPENDENCIES

#### Description:

OpenZeppelin contracts may be considered draft contracts if they have not received adequate security auditing or are liable to change with future development. These contracts are still a draft and are not considered ready for mainnet use.

#### **Proof Of Concept**

```solidity
File: contracts/token/ArcadeToken.sol

7: import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";

```

### [L-5] INSUFFICIENT COVERAGE

#### Description:

The test coverage rate of the project is 99%. Testing all functions is best practice in terms of security criteria.

### [L-6] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: contracts/ARCDVestingVault.sol

330:         uint256 unlocked = grant.cliffAmount + (postCliffAmount * blocksElapsedSinceCliff) / totalBlocksPostCliff;

```

```solidity
File: contracts/NFTBoostVault.sol

494:         uint128 newVotingPower = (_amount * multiplier) / MULTIPLIER_DENOMINATOR;

633:             return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;

```

```solidity
File: contracts/token/ArcadeToken.sol

154:         uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;

```

#### Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

### [L-7] OWNER CAN RENOUNCE OWNERSHIP

#### Description:

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

#### **Proof Of Concept**

```solidity
File: contracts/nft/BadgeDescriptor.sol

5: import "@openzeppelin/contracts/access/Ownable.sol";

17: contract BadgeDescriptor is IBadgeDescriptor, Ownable {

```

```solidity
File: contracts/token/ArcadeTokenDistributor.sol

5: import "@openzeppelin/contracts/access/Ownable.sol";

23: contract ArcadeTokenDistributor is Ownable {

```

#### Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

### [L-8] THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS

#### Description:

If a pair gets created and after a while one of the tokens gets self-destroyed (maybe because of a bug) then `safeTransfer` would still succeed. It’s probably a good idea to check if the contract still exists by checking the bytecode length.

#### **Proof Of Concept**

```solidity
File: contracts/ARCDVestingVault.sol

167:         token.safeTransfer(who, withdrawable);

172:         token.safeTransfer(msg.sender, remaining);

217:         token.safeTransfer(recipient, amount);

252:         token.safeTransfer(msg.sender, withdrawable);

```

```solidity
File: contracts/ArcadeTreasury.sol

369:             IERC20(token).safeTransfer(destination, amount);

```

```solidity
File: contracts/NFTBoostVault.sol

258:         token.safeTransfer(msg.sender, amount);

```

```solidity
File: contracts/token/ArcadeAirdrop.sol

67:         token.safeTransfer(destination, unclaimed);

```

```solidity
File: contracts/token/ArcadeTokenDistributor.sol

79:         arcadeToken.safeTransfer(_treasury, treasuryAmount);

95:         arcadeToken.safeTransfer(_devPartner, devPartnerAmount);

111:         arcadeToken.safeTransfer(_communityRewards, communityRewardsAmount);

127:         arcadeToken.safeTransfer(_communityAirdrop, communityAirdropAmount);

144:         arcadeToken.safeTransfer(_vestingTeam, vestingTeamAmount);

161:         arcadeToken.safeTransfer(_vestingPartner, vestingPartnerAmount);

```

### [L-9] STACK TOO DEEP WHEN COMPILING

#### Description:

The project cannot be compiled due to the “stack too deep” error.

The “stack too deep” error is a limitation of the current code generator. The EVM stack only has 16 slots and that’s sometimes not enough to fit all the local variables, parameters and/or return variables. The solution is to move some of them to memory, which is more expensive but at least makes your code compile.

ref: https://forum.openzeppelin.com/t/stack-too-deep-when-compiling-inline-assembly/11391/6

#### **Proof Of Concept**

```solidity
File: contracts/token/ArcadeToken.sol

7: import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";

```

### [L-10] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

As for block.number, considering the block time on Ethereum is generally about 14 seconds, it`s possible to predict the time delta between blocks. However, block times are not constant and are subject to change for a variety of reasons, e.g. fork reorganisations and the difficulty bomb. Due to variable block times, block.number should also not be relied on for precise calculations of time.

Reference: https://swcregistry.io/docs/SWC-116
Reference: https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md

#### **Proof Of Concept**

```solidity
File: contracts/ARCDVestingVault.sol

106:             startTime = uint128(block.number);

234:         if (grant.cliff > block.number) revert AVV_CliffNotReached(grant.cliff);

319:         if (block.number < grant.cliff) {

323:         if (block.number >= grant.expiration) {

328:         uint256 blocksElapsedSinceCliff = block.number - grant.cliff;

```

```solidity
File: contracts/ArcadeTreasury.sol

308:         if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {

309:             revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);

319:         lastAllowanceSet[token] = uint48(block.timestamp);

360:         uint256 spentThisBlock = blockExpenditure[block.number];

362:         blockExpenditure[block.number] = amount + spentThisBlock;

386:         uint256 spentThisBlock = blockExpenditure[block.number];

388:         blockExpenditure[block.number] = amount + spentThisBlock;

```

```solidity
File: contracts/BaseVotingVault.sol

54:         if (_staleBlockLag >= block.number) revert BVV_UpperLimitBlock(_staleBlockLag);

101:         return votingPower.findAndClear(user, blockNumber, block.number - staleBlockLag);

```

```solidity
File: contracts/nft/ReputationBadge.sol

108:         if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));

108:         if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));

146:             if (_claimData[i].claimExpiration <= block.timestamp) {

```

```solidity
File: contracts/token/ArcadeAirdrop.sol

63:         if (block.timestamp <= expiration) revert AA_ClaimingNotExpired();

```

```solidity
File: contracts/token/ArcadeToken.sol

119:         mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;

146:         if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);

146:         if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);

151:         mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;

```

### [L-11] Account existence check for low-level calls

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-callsn

#### **Proof Of Concept**

```solidity
File: contracts/ArcadeTreasury.sol

341:             (bool success, ) = targets[i].call(calldatas[i]);

```

#### Recommended Mitigation Steps:

In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

### [L-12] UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT

#### Description:

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert

#### **Proof Of Concept**

```solidity
File: contracts/ArcadeTreasury.sol

397:     receive() external payable {}

```

#### Recommended Mitigation Steps:

The function should call another function, otherwise it should revert

### [L-13] USE `SAFETRANSFER` INSTEAD OF `TRANSFER`

#### Description:

It is good to add a `require()` statement that checks the return value of token transfers or to use something like OpenZeppelin’s `safeTransfer`/`safeTransferFrom` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)‘s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

#### **Proof Of Concept**

```solidity
File: contracts/ArcadeTreasury.sol

367:             payable(destination).transfer(amount);

```

```solidity
File: contracts/nft/ReputationBadge.sol

171:         payable(recipient).transfer(balance);

```

#### Recommended Mitigation Steps:

Consider using `safeTransfer`/`safeTransferFrom` or `require()` consistently.

### [L-14] NOT USING THE LATEST VERSION OF OPENZEPPELIN FROM DEPENDENCIES

#### Description:

The package.json configuration file says that the project is using 4.3.2 of OpenZeppelin which has a not last update version.

Patched versions for @openzeppelin/contracts is 4.4.1.

That is why @openzeppelin/contracts version 4.3.2 is vulnerable.

https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-9c22-pwxw-p6hx

#### **Proof Of Concept**

```solidity
File: package.json

103:  "@openzeppelin/contracts": "4.3.2",

```

#### Recommended Mitigation Steps:

Use patched versions. Latest non vulnerable version 4.4.1.
