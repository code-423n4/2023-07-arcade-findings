## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#gas1-consider-activating-viair-for-deploying) | Consider activating `via-ir` for deploying | 1 | 250 |
| [GAS&#x2011;2](#gas2-counting-down-in-for-statements-is-more-gas-efficient) | Counting down in `for` statements is more gas efficient | 3 | 771 |
| [GAS&#x2011;3](#gas3-multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache) | Multiple accesses of a mapping/array should use a local variable cache | 11 | 880 |
| [GAS&#x2011;4](#gas4-the-result-of-a-function-call-should-be-cached-rather-than-recalling-the-function) | The result of a function call should be cached rather than re-calling the function | 2 | 100 |
| [GAS&#x2011;5](#gas5-use-nested-if-and-avoid-multiple-check-combinations) | Use nested `if` and avoid multiple check combinations | 5 | 30 |
| [GAS&#x2011;6](#gas6-use-solidity-version-0820-to-gain-some-gas-boost) | Use the latest solidity (prior to 0.8.20 if on L2s) to gain some gas boost | 12 | 1056 |
| [GAS&#x2011;7](#gas7-using-xor-^-and-and-&-bitwise-equivalents) | Using XOR (^) and AND (&) bitwise equivalents | 64 | 832 |
| [GAS&#x2011;8](#gas8-using-thisfn-wastes-gas) | Using `this.<fn>()` wastes gas | 1 | 100 |

Total: 103 contexts over 8 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Consider activating `via-ir` for deploying

The IR-based code generator was introduced with an aim to not only allow code generation to be more transparent and auditable but also to enable more powerful optimization passes that span across functions.

You can enable it on the command line using `--via-ir` or with the option `{"viaIR": true}`.

This will take longer to compile, but you can just simple test it before deploying and if you got a better benchmark then you can add --via-ir to your deploy command

More on: https://docs.soliditylang.org/en/v0.8.17/ir-breaking-changes.html



### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

```solidity
File: ArcadeTreasury.sol

339: for (uint256 i = 0; i < targets.length; ++i) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L339

```solidity
File: NFTBoostVault.sol

345: for (uint256 i = 0; i < userAddresses.length; ++i) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L345

```solidity
File: ReputationBadge.sol

144: for (uint256 i = 0; i < _claimData.length; i++) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L144




#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |




### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: ArcadeTreasury.sol

281: if (thresholds.small < gscAllowance[token]) {
282: gscAllowance[token] = thresholds.small;

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L281-L282

```solidity
File: ArcadeTreasury.sol

308: if (uint48(block.timestamp) < lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN) {
309: revert T_CoolDownPeriod(block.timestamp, lastAllowanceSet[token] + SET_ALLOWANCE_COOL_DOWN);
319: lastAllowanceSet[token] = uint48(block.timestamp);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L308

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L309

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L319



```solidity
File: ArcadeTreasury.sol

360: uint256 spentThisBlock = blockExpenditure[block.number];
386: uint256 spentThisBlock = blockExpenditure[block.number];
362: blockExpenditure[block.number] = amount + spentThisBlock;
388: blockExpenditure[block.number] = amount + spentThisBlock;

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L360

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L386

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L362

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L388




```solidity
File: ReputationBadge.sol

111: if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
116: amountClaimed[recipient][tokenId] += amount;

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L111

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L116





</details>




### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following:

#### <ins>Proof Of Concept</ins>


```solidity
File: ArcadeToken.sol

154: uint256 mintCapAmount = (totalSupply() * MINT_CAP) / PERCENT_DENOMINATOR;
156: revert AT_MintingCapExceeded(totalSupply(), mintCapAmount, _amount);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L154

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L156






### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Use nested `if` and avoid multiple check combinations

Using nested `if`, is cheaper than using `&&` multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: NFTBoostVault.sol

247: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L247

```solidity
File: NFTBoostVault.sol

317: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L317

```solidity
File: NFTBoostVault.sol

472: if (_tokenAddress != address(0) && _tokenId != 0) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L472

```solidity
File: NFTBoostVault.sol

632: if (registration.tokenAddress != address(0) && registration.tokenId != 0) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L632

```solidity
File: NFTBoostVault.sol

659: if (tokenAddress != address(0) && tokenId != 0) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L659



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }
    
    contract Contract0 {
    
        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }
    
    }
    
    contract Contract1 {
    
        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |



### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Use the latest solidity (prior to 0.8.20 if on L2s) to gain some gas boost

Upgrade to the latest solidity version 0.8.20 to get additional gas savings.
See latest release for reference: https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: ArcadeGSCCoreVoting.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeGSCCoreVoting.sol#L3

```solidity
File: ArcadeGSCVault.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeGSCVault.sol#L3

```solidity
File: ArcadeTreasury.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L3

```solidity
File: ARCDVestingVault.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L3

```solidity
File: BaseVotingVault.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L3

```solidity
File: ImmutableVestingVault.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ImmutableVestingVault.sol#L3

```solidity
File: NFTBoostVault.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L3

```solidity
File: BadgeDescriptor.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/BadgeDescriptor.sol#L3

```solidity
File: ReputationBadge.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L3

```solidity
File: ArcadeAirdrop.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeAirdrop.sol#L3

```solidity
File: ArcadeToken.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L3

```solidity
File: ArcadeTokenDistributor.sol

pragma solidity 0.8.18;
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L3



</details>




### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: ArcadeTreasury.sol

113: if (destination == address(0)) revert T_ZeroAddress("destination");
114: if (amount == 0) revert T_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L113-L114

```solidity
File: ArcadeTreasury.sol

135: if (destination == address(0)) revert T_ZeroAddress("destination");
136: if (amount == 0) revert T_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L135-L136

```solidity
File: ArcadeTreasury.sol

154: if (destination == address(0)) revert T_ZeroAddress("destination");
155: if (amount == 0) revert T_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L154-L155

```solidity
File: ArcadeTreasury.sol

173: if (destination == address(0)) revert T_ZeroAddress("destination");
174: if (amount == 0) revert T_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L173-L174

```solidity
File: ArcadeTreasury.sol

194: if (spender == address(0)) revert T_ZeroAddress("spender");
195: if (amount == 0) revert T_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L194-L195

```solidity
File: ArcadeTreasury.sol

216: if (spender == address(0)) revert T_ZeroAddress("spender");
217: if (amount == 0) revert T_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L216-L217

```solidity
File: ArcadeTreasury.sol

235: if (spender == address(0)) revert T_ZeroAddress("spender");
236: if (amount == 0) revert T_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L235-L236

```solidity
File: ArcadeTreasury.sol

254: if (spender == address(0)) revert T_ZeroAddress("spender");
255: if (amount == 0) revert T_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L254-L255

```solidity
File: ArcadeTreasury.sol

271: if (token == address(0)) revert T_ZeroAddress("token");
273: if (thresholds.small == 0) revert T_ZeroAmount();
276: if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L271

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L273

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L276



```solidity
File: ArcadeTreasury.sol

304: if (token == address(0)) revert T_ZeroAddress("token");
305: if (newAllowance == 0) revert T_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L304-L305

```solidity
File: ARCDVestingVault.sol

101: if (who == address(0)) revert AVV_ZeroAddress("who");
102: if (amount == 0) revert AVV_InvalidAmount();
105: if (startTime == 0) {
109: if (cliff >= expiration || cliff < startTime) revert AVV_InvalidSchedule();
125: delegatee = delegatee == address(0) ? who : delegatee;

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L101

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L102

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L105

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L109

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L125



```solidity
File: ARCDVestingVault.sol

162: if (grant.allocation == 0) revert AVV_NoGrantSet();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L162

```solidity
File: ARCDVestingVault.sol

229: if (amount == 0) revert AVV_InvalidAmount();
233: if (grant.allocation == 0) revert AVV_NoGrantSet();
241: if (amount == withdrawable) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L229

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L233

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L241



```solidity
File: ARCDVestingVault.sol

262: if (to == grant.delegatee) revert AVV_AlreadyDelegated();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L262

```solidity
File: BaseVotingVault.sol

69: if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L69

```solidity
File: BaseVotingVault.sol

81: if (manager_ == address(0)) revert BVV_ZeroAddress("manager");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L81

```solidity
File: NFTBoostVault.sol

120: if (amount == 0) revert NBV_ZeroAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L120

```solidity
File: NFTBoostVault.sol

144: if (amount == 0) revert NBV_ZeroAmount();
145: if (user == address(0)) revert NBV_ZeroAddress("user");
152: if (registration.delegatee == address(0)) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L144

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L145

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L152



```solidity
File: NFTBoostVault.sol

183: if (to == address(0)) revert NBV_ZeroAddress("to");
188: if (to == registration.delegatee) revert NBV_AlreadyDelegated();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L183

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L188



```solidity
File: NFTBoostVault.sol

225: if (amount == 0) revert NBV_ZeroAmount();
246: if (registration.withdrawn == registration.amount) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L225

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L246



```solidity
File: NFTBoostVault.sol

267: if (amount == 0) revert NBV_ZeroAmount();
273: if (registration.delegatee == address(0)) revert NBV_NoRegistration();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L267

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L273



```solidity
File: NFTBoostVault.sol

306: if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);
314: if (registration.delegatee == address(0)) revert NBV_NoRegistration();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L306

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L314



```solidity
File: NFTBoostVault.sol

422: if (tokenAddress == address(0) || tokenId == 0) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L422

```solidity
File: NFTBoostVault.sol

477: if (multiplier == 0) revert NBV_NoMultiplierSet();
491: _delegatee = _delegatee == address(0) ? user : _delegatee;

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L477

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L491



```solidity
File: NFTBoostVault.sol

552: if (registration.tokenAddress == address(0) || registration.tokenId == 0)

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L552

```solidity
File: NFTBoostVault.sol

588: if (change == 0) return;

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L588

```solidity
File: NFTBoostVault.sol

611: if (registration.withdrawn == registration.amount) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L611

```solidity
File: ReputationBadge.sol

141: if (_claimData.length == 0) revert RB_NoClaimData();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L141

```solidity
File: ReputationBadge.sol

164: if (recipient == address(0)) revert RB_ZeroAddress("recipient");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L164

```solidity
File: ReputationBadge.sol

185: if (_descriptor == address(0)) revert RB_ZeroAddress("descriptor");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L185

```solidity
File: ArcadeAirdrop.sol

64: if (destination == address(0)) revert AA_ZeroAddress("destination");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeAirdrop.sol#L64

```solidity
File: ArcadeToken.sol

133: if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L133

```solidity
File: ArcadeToken.sol

147: if (_to == address(0)) revert AT_ZeroAddress("to");
148: if (_amount == 0) revert AT_ZeroMintAmount();

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L147-L148

```solidity
File: ArcadeTokenDistributor.sol

75: if (_treasury == address(0)) revert AT_ZeroAddress("treasury");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L75

```solidity
File: ArcadeTokenDistributor.sol

91: if (_devPartner == address(0)) revert AT_ZeroAddress("devPartner");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L91

```solidity
File: ArcadeTokenDistributor.sol

107: if (_communityRewards == address(0)) revert AT_ZeroAddress("communityRewards");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L107

```solidity
File: ArcadeTokenDistributor.sol

123: if (_communityAirdrop == address(0)) revert AT_ZeroAddress("communityAirdrop");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L123

```solidity
File: ArcadeTokenDistributor.sol

140: if (_vestingTeam == address(0)) revert AT_ZeroAddress("vestingTeam");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L140

```solidity
File: ArcadeTokenDistributor.sol

157: if (_vestingPartner == address(0)) revert AT_ZeroAddress("vestingPartner");

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L157



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |




### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Using `this.<fn>()` wastes gas

Calling an external function internally, through the use of `this` wastes the gas overhead of calling an external function (100 gas).

#### <ins>Proof Of Concept</ins>

```solidity
File: NFTBoostVault.sol

698: return this.onERC1155Received.selector;

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L698






