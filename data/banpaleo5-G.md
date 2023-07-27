# Table of Contents
| Number | Issues Details                                                                                         | Count |
| :----- | :----------------------------------------------------------------------------------------------------- | :---- |
| [G-1] | When it's impossible for them to overflow, such as in for- and while-loops, `++i` and `i++` should be written as `unchecked{++i}` and `unchecked{i++}` respectively for better efficiency | 2 |
| [G-2] | Avoid repeatedly accessing the length property of an array within each iteration of a for-loop | 2 |
| [G-3] | Designating the `constructor` as `payable` saves gas | 4 |
| [G-4] | Employing the `delete` statement can contribute to gas conservation | 3 |
| [G-5] | Recommended to utilize OpenZeppelin contracts version v4.9.0 for gas efficiency | 1 |
| [G-6] | Either remove empty blocks or have them emit an event, as they serve no purpose as is | 1 |
| [G-7] | Using `++i` is more gas-efficient than `i++` within loops | 1 |
| [G-8] | Employing `uints` or `ints` that are smaller than 32 bytes (256 bits) leads to overhead costs | 2 |
| [G-9] | The operation `<x> += <y>` consumes more gas than `<x> = <x> + <y>` | 3 |
| [G-10] | Declaring constants as `private` instead of `public` leads to gas savings | 1 |


## [G-1]</a><a name="G-1"> When it's impossible for them to overflow, such as in for- and while-loops, `++i` and `i++` should be written as `unchecked{++i}` and `unchecked{i++}` respectively for better efficiency

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
for (uint256 i = 0; i < userAddresses.length; ++i) {
```

```solidity
File: ReputationBadge.sol
for (uint256 i = 0; i < _claimData.length; i++) {
```
</details>



## [G-2]</a><a name="G-2"> Avoid repeatedly accessing the length property of an array within each iteration of a for-loop

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
for (uint256 i = 0; i < userAddresses.length; ++i) {
```

```solidity
File: ReputationBadge.sol
for (uint256 i = 0; i < _claimData.length; i++) {
```
</details>



## [G-3]</a><a name="G-3"> Designating the `constructor` as `payable` saves gas

<details>
<summary><i>There are 4 instances of this issue:</i></summary>

```solidity
File: ArcadeAirdrop.sol
constructor(
        address _governance,
        bytes32 _merkleRoot,
        IERC20 _token,
        uint256 _expiration,
        INFTBoostVault _votingVault
    ) ArcadeMerkleRewards(_merkleRoot, _token, _expiration, _votingVault)
```

```solidity
File: BaseVotingVault.sol
constructor(IERC20 _token, uint256 _staleBlockLag)
```

```solidity
File: ReputationBadge.sol
constructor(address _owner, address _descriptor) ERC1155("")
```

```solidity
File: NFTBoostVault.sol
constructor(
        IERC20 token,
        uint256 staleBlockLag,
        address timelock,
        address manager
    ) BaseVotingVault(token, staleBlockLag)
```
</details>



## [G-4]</a><a name="G-4"> Employing the `delete` statement can contribute to gas conservation

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
registration.withdrawn = 0;
```

```solidity
File: NFTBoostVault.sol
registration.amount = 0;
registration.latestVotingPower = 0;
registration.withdrawn = 0;
```

```solidity
File: NFTBoostVault.sol
registration.tokenId = 0;
```
</details>



## [G-5]</a><a name="G-5"> Recommended to utilize OpenZeppelin contracts version v4.9.0 for gas efficiency

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: package.json
    "@openzeppelin/contracts": "4.3.2"
```

```solidity
File: package.json
    "@openzeppelin/contracts-upgradeable": "^4.7.2"
```
</details>



## [G-6]</a><a name="G-6"> Either remove empty blocks or have them emit an event, as they serve no purpose as is

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ArcadeTreasury.sol
receive() external payable {}
```
</details>



## [G-7]</a><a name="G-7"> Using `++i` is more gas-efficient than `i++` within loops

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ReputationBadge.sol
for (uint256 i = 0; i < _claimData.length; i++) {
```
</details>



## [G-8]</a><a name="G-8"> Employing `uints` or `ints` that are smaller than 32 bytes (256 bits) leads to overhead costs

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: ArcadeTreasury.sol
uint48 public constant SET_ALLOWANCE_COOL_DOWN = 7 days;
```

```solidity
File: ArcadeToken.sol
uint48 public constant MIN_TIME_BETWEEN_MINTS = 365 days;
```
</details>



## [G-9]</a><a name="G-9"> The operation `<x> += <y>` consumes more gas than `<x> = <x> + <y>`

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: NFTBoostVault.sol
balance.data += _amount;
```

```solidity
File: ARCDVestingVault.sol
grant.withdrawn += uint128(withdrawable);
grant.withdrawn += uint128(amount);
```

```solidity
File: NFTBoostVault.sol
balance.data += amount;
registration.amount += amount;
```
</details>



## [G-10]</a><a name="G-10"> Declaring constants as `private` instead of `public` leads to gas savings

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: ArcadeToken.sol
uint48 public constant MIN_TIME_BETWEEN_MINTS = 365 days;
uint256 public constant MINT_CAP = 2;
uint256 public constant PERCENT_DENOMINATOR = 100;
uint256 public constant INITIAL_MINT_AMOUNT = 100_000_000 ether;
```
</details>