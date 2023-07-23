## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol

```solidity
// place this event after state variables
20:    event SetBaseURI(address indexed caller, string baseURI);
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol

```solidity
// place this modifier before the constructor
167:    modifier onlyMinter() {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol

```solidity
// place this modifier before the constructor
701:    modifier onlyAirdrop() {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol

```solidity
// place this modifier before the constructor
701:    modifier onlyAirdrop() {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol

```solidity
// place these modifiers before the constructor
187:    modifier onlyManager() {
196:    modifier onlyTimelock() {
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol

```solidity
// place this non-view / non-pure before all the external view/pure
57:    function setBaseURI(string memory newBaseURI) external onlyOwner {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol

```solidity
// external functions coming after public one.
140:    function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {
163:    function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {

// public function coming after internal ones.
217:    function supportsInterface(
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol

```solidity
// external functions coming after public ones
378:    function unlock() external override onlyTimelock {
392:    function setAirdropContract(address newAirdropContract) external override onlyManager {
436:    function getRegistration(address who) external view override returns (NFTBoostVaultStorage.Registration memory) {
445:    function getAirdropContract() external view override returns (address) {

// place these non-view / non-pure before all the internal view/pure
548:    function _withdrawNft() internal {
579:    function _syncVotingPower(address who, NFTBoostVaultStorage.Registration storage registration) internal {
650:    function _lockTokens(
673:    function _lockNft(address from, address tokenAddress, uint128 tokenId, uint128 nftAmount) internal {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol

```solidity
// external functions coming after some public ones
392:    function setAirdropContract(address newAirdropContract) external override onlyManager {
436:    function getRegistration(address who) external view override returns (NFTBoostVaultStorage.Registration memory) {
445:    function getAirdropContract() external view override returns (address) {

// place these non-view / non-pure before all the internal view/pure
650:    function _lockTokens(
673:    function _lockNft(address from, address tokenAddress, uint128 tokenId, uint128 nftAmount) internal {

// public function coming after internal ones
697:    function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol

```solidity
// place this non-view / non-pure before all the internal view/pure
341:    function _syncVotingPower(address who, ARCDVestingVaultStorage.Grant storage grant) internal {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol

```solidity
// place this `receive` right after the constructor
397:    receive() external payable {}
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol

```solidity
26:    event SetMerkleRoot(bytes32 indexed merkleRoot);
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol

```solidity
// @params missing
697:    function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {

701:    modifier onlyAirdrop() {
```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ImmutableVestingVault.sol

```solidity
// @param missing
40:    function revokeGrant(address) public pure override {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol

```solidity
// constant state variables should be UPPER_CASE
32:    uint256 public constant treasuryAmount = 25_500_000 ether;
37:    uint256 public constant devPartnerAmount = 600_000 ether;
42:    uint256 public constant communityRewardsAmount = 15_000_000 ether;
47:    uint256 public constant communityAirdropAmount = 10_000_000 ether;
52:    uint256 public constant vestingTeamAmount = 16_200_000 ether;
57:    uint256 public constant vestingPartnerAmount = 32_700_000 ether;
```
