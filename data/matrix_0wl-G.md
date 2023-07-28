## Gas Optimizations

|        | Issue                                                                                                                   |
| ------ | :---------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables                   |
| GAS-2  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                                             |
| GAS-3  | <ARRAY>.LENGTH SHOULD NOT BE LOOKED UP IN EVERY LOOP OF A FOR-LOOP                                                      |
| GAS-4  | USE FUNCTION INSTEAD OF MODIFIERS                                                                                       |
| GAS-5  | IT COSTS MORE GAS TO INITIALIZE NON-CONSTANT/NON-IMMUTABLE VARIABLES TO ZERO THAN TO LET THE DEFAULT OF ZERO BE APPLIED |
| GAS-6  | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT                                |
| GAS-7  | KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE                                             |
| GAS-8  | USING PRIVATE RATHER THAN PUBLIC FOR CONSTANTS, SAVES GAS                                                               |
| GAS-9  | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) = for-loop and while-loops  |
| GAS-10 | The increment in for loop postcondition can be made unchecked                                                           |
| GAS-11 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                                     |
| GAS-12 | Use != 0 instead of > 0 for unsigned integer comparison                                                                 |
| GAS-13 | USE BYTES32 INSTEAD OF STRING                                                                                           |

### [GAS-1] `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables

#### Description:

Using the addition operator instead of plus-equals saves gas

#### **Proof Of Concept**

```solidity
File: contracts/ARCDVestingVault.sol

141:         unassigned.data -= amount;

166:         grant.withdrawn += uint128(withdrawable);

171:         grant.withdrawn += uint128(remaining);

200:         unassigned.data += amount;

215:         unassigned.data -= amount;

242:             grant.withdrawn += uint128(withdrawable);

244:             grant.withdrawn += uint128(amount);

```

```solidity
File: contracts/ArcadeTreasury.sol

117:         gscAllowance[token] -= amount;

198:         gscAllowance[token] -= amount;

```

```solidity
File: contracts/NFTBoostVault.sol

161:             balance.data += amount;

164:             registration.amount += amount;

239:         balance.data -= amount;

241:         registration.withdrawn += amount;

278:         balance.data += amount;

281:         registration.amount += amount;

505:         balance.data += _amount;

```

```solidity
File: contracts/nft/ReputationBadge.sol

116:         amountClaimed[recipient][tokenId] += amount;

```

### [GAS-2] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything.

#### **Proof Of Concept**

```solidity
File: contracts/ARCDVestingVault.sol

201:         token.transferFrom(msg.sender, address(this), amount);

```

```solidity
File: contracts/ArcadeTreasury.sol

367:             payable(destination).transfer(amount);

```

```solidity
File: contracts/NFTBoostVault.sol

657:         token.transferFrom(from, address(this), amount);

```

```solidity
File: contracts/nft/ReputationBadge.sol

171:         payable(recipient).transfer(balance);

```

### [GAS-3] <ARRAY>.LENGTH SHOULD NOT BE LOOKED UP IN EVERY LOOP OF A FOR-LOOP

#### Description:

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

[Source](https://code4rena.com/reports/2022-12-backed#g14--arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)

#### **Proof Of Concept**

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

### [GAS-4] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: contracts/BaseVotingVault.sol

187:     modifier onlyManager() {

196:     modifier onlyTimelock() {

```

```solidity
File: contracts/NFTBoostVault.sol

701:     modifier onlyAirdrop() {

```

```solidity
File: contracts/token/ArcadeToken.sol

167:     modifier onlyMinter() {

```

### [GAS-5] IT COSTS MORE GAS TO INITIALIZE NON-CONSTANT/NON-IMMUTABLE VARIABLES TO ZERO THAN TO LET THE DEFAULT OF ZERO BE APPLIED

#### Description:

If a variable is not set/initialized, the default value is assumed (0, false, 0x0 … depending on the data type). You are simply wasting gas if you directly initialize it with its default value.

#### **Proof Of Concept**

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

### [GAS-6] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

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

```

### [GAS-7] KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE

#### **Proof Of Concept**

```solidity
File: contracts/ArcadeTreasury.sol

48:     bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");

```

```solidity
File: contracts/nft/ReputationBadge.sol

44:     bytes32 public constant ADMIN_ROLE = keccak256("ADMIN");

```

### [GAS-8] USING PRIVATE RATHER THAN PUBLIC FOR CONSTANTS, SAVES GAS

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple. There are four instances of public constants.

#### **Proof Of Concept**

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

### [GAS-9] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) = for-loop and while-loops

#### Description:

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

#### **Proof Of Concept**

```solidity
File: contracts/nft/ReputationBadge.sol

144:         for (uint256 i = 0; i < _claimData.length; i++) {

```

### [GAS-10] The increment in for loop postcondition can be made unchecked

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.

The for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks.One can manually do this.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: contracts/nft/ReputationBadge.sol

144:         for (uint256 i = 0; i < _claimData.length; i++) {

```

### [GAS-11] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
Note that this optimization seems to be dependent on usage of a more recent Solidity version. The following gas savings are on version 0.8.17.

#### **Proof Of Concept**

```solidity
File: contracts/ArcadeTreasury.sol

365:    if (address(token) == ETH_CONSTANT) {
366:            // will out-of-gas revert if recipient is a contract with logic inside receive()
367:            payable(destination).transfer(amount);
368:        } else {
369:            IERC20(token).safeTransfer(destination, amount);
370:        }

```

```solidity
File: contracts/NFTBoostVault.sol

589:    if (change > 0) {
590:            votingPower.push(registration.delegatee, delegateeVotes + uint256(change));
591:        } else {
592:            // if the change is negative, we multiply by -1 to avoid underflow when casting
593:            votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));
594:        }

```

### [GAS-12] Use != 0 instead of > 0 for unsigned integer comparison

#### Description:

To update this with using at least 0.8.6 there is no difference in gas usage with != 0 or > 0.
[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: contracts/NFTBoostVault.sol

589:         if (change > 0) {

```

```solidity
File: contracts/nft/BadgeDescriptor.sol

49:         return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

```

### [GAS-13] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: contracts/nft/BadgeDescriptor.sol

20:     event SetBaseURI(address indexed caller, string baseURI);

24:     string public baseURI;

34:     constructor(string memory _baseURI) {

48:     function tokenURI(uint256 tokenId) external view override returns (string memory) {

49:         return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

57:     function setBaseURI(string memory newBaseURI) external onlyOwner {

```

```solidity
File: contracts/nft/ReputationBadge.sol

129:     function uri(uint256 tokenId) public view override(ERC1155, IReputationBadge) returns (string memory) {

```
