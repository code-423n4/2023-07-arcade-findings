## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | _setupRole() is deprecated by openzeppelin | 2 |
| [L&#x2011;02] | Calls inside loops that may address DoS | 1 |
| [L&#x2011;03] | gscSpend() does not check amount validation per Natspec comment | 1 |
| [L&#x2011;04] | draft-ERC20Permit.sol is deprecated by openzeppelin | 1 |
| [L&#x2011;05] | unsafe casting in ARCDVestingVault.sol | 3 |
| [L&#x2011;06] | gscApprove() does not check amount validation per Natspec comment | 1 |
| [L&#x2011;07] | _spend() does not destination address could be contract address with no receive() | 1 |
| [L&#x2011;08] | Missing event in setTimelock() and setManager() | 2 |
| [L&#x2011;09] | staleBlockLag should not be immutable because of the ever changing block formation duration | 2 |
| [L&#x2011;10] | address(0) can be delegated in delegate() | 1 |
| [L&#x2011;11] | Any tokens directly sent to ARCDVestingVault.sol will be permanently locked | 1 |
| [L&#x2011;12] | Avoid setting null _merkleRoot in setMerkleRoot() | 1 |
| [L&#x2011;13] | Use latest version of openzeppelin library for mitigating MerkleProof.sol bug | 1 |
| [L&#x2011;14] | Misleading comment on upgradeable contracts | 1 |


### [L&#x2011;01]  _setupRole() is deprecated by openzeppelin
In ArcadeTreasury.sol and ReputationBadge.sol, The constructor has used the _setupRole() function to set the roles in constructor. This function is inherited from openzeppelin AccessControl.sol but openzeppelin has deprecated _setupRole() function in favor of _grantRole().

Reference link: https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3488

There are 2 instance of this issue i.e in ArcadeTreasury.sol and ReputationBadge.sol constructor.

### Recommended Mitigation Steps
Use _grantRole() instead of deprecated _setupRole().

**Instance 1:**

```Solidity
File: contracts/ArcadeTreasury.sol

    constructor(address _timelock) {
        if (_timelock == address(0)) revert T_ZeroAddress("timelock");

-        _setupRole(ADMIN_ROLE, _timelock);
+        _grantRole(ADMIN_ROLE, _timelock);
        _setRoleAdmin(ADMIN_ROLE, ADMIN_ROLE);
        _setRoleAdmin(GSC_CORE_VOTING_ROLE, ADMIN_ROLE);
        _setRoleAdmin(CORE_VOTING_ROLE, ADMIN_ROLE);
    }
```

**Instance 2:**

```Solidity
File: contracts/nft/ReputationBadge.sol

    constructor(address _owner, address _descriptor) ERC1155("") {
        if (_owner == address(0)) revert RB_ZeroAddress("owner");
        if (_descriptor == address(0)) revert RB_ZeroAddress("descriptor");

-        _setupRole(ADMIN_ROLE, _owner);
+        _grantRole(ADMIN_ROLE, _owner);
        _setRoleAdmin(ADMIN_ROLE, ADMIN_ROLE);
        _setRoleAdmin(BADGE_MANAGER_ROLE, ADMIN_ROLE);
        _setRoleAdmin(RESOURCE_MANAGER_ROLE, ADMIN_ROLE);

        descriptor = IBadgeDescriptor(_descriptor);
    }
```

### [L&#x2011;02]  Calls inside loops that may address DoS
In ArcadeTreasury.sol contract batchCalls() function,
Calls to external contracts inside a loop are dangerous because it could lead to DoS if one of the calls reverts or execution runs out of gas. Such issue also introduces chance of problems with the gas limits.

**Per SWC-113:**
External calls can fail accidentally or deliberately, which can cause a DoS condition in the contract. To minimize the damage caused by such failures, it is better to isolate each external call into its own transaction that can be initiated by the recipient of the call.

Reference link- https://swcregistry.io/docs/SWC-113

**Code Reference:**
```Solidity
File: contracts/ArcadeTreasury.sol

333    function batchCalls(
334        address[] memory targets,
345        bytes[] calldata calldatas
346    ) external onlyRole(ADMIN_ROLE) nonReentrant {
347        if (targets.length != calldatas.length) revert T_ArrayLengthMismatch();
348        // execute a package of low level calls
349        for (uint256 i = 0; i < targets.length; ++i) {
350            if (spendThresholds[targets[i]].small != 0) revert T_InvalidTarget(targets[i]);
351            (bool success, ) = targets[i].call(calldatas[i]);
352            // revert if a single call fails
353            if (!success) revert T_CallFailed();
354        }
355    }
```

### Recommended Mitigation Steps
1) Avoid combining multiple calls in a single transaction, especially when calls are executed as part of a loop
2) Always assume that external calls can fail
3) Implement the contract logic to handle failed calls

### [L&#x2011;03]  gscSpend() does not check amount validation per Natspec comment
In ArcadeTreasury.sol, gscSpend() Natspec comment has some required validation before the amount is passed to spend by gsc. The comment states,

```Solidity
101     * @notice function for the GSC to spend tokens from the treasury. The amount to be
102     *         spent must be less than or equal to the GSC's allowance for that specific token.
```
However, there is no such validation to amount in gscSpend().

There is 1 instance of this issue:

### Recommended Mitigation steps
```Solidity
File: contracts/ArcadeTreasury.sol

    function gscSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
        if (destination == address(0)) revert T_ZeroAddress("destination");
        if (amount == 0) revert T_ZeroAmount();
+       require(amount <= gscAllowance[token], "invalid allowance amount");

        // Will underflow if amount is greater than remaining allowance
        gscAllowance[token] -= amount;

        _spend(token, amount, destination, spendThresholds[token].small);
    }
```
### [L&#x2011;04]  _setupRole() is deprecated by openzeppelin
In ArcadeToken.sol, draft-ERC20Permit.sol is used in contract but Openzeppelin has deprecated the use of draft-ERC20Permit.sol in v4.9.0 release. Check out the deprecations [here](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0).

There is 1 instance of this issue:

### Recommended Mitigation steps
Use [ERC20Permit.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Permit.sol).

### [L&#x2011;05]  unsafe casting in ARCDVestingVault.sol
In ARCDVestingVault.sol, There is unsafe down casting from uint256 to uint128, uint256 to int256, etc. resulting in accounting errors leading loss of user amount in case of claim, etc. Below functions highlights the unsafe down casting issues with recommendations,

revokeGrant(), 

```Solidity
File: contracts/ARCDVestingVault.sol

157    function revokeGrant(address who) external virtual onlyManager {

       // Some code

165        uint256 withdrawable = _getWithdrawableAmount(grant);
166        grant.withdrawn += uint128(withdrawable);


       // Some code

170        uint256 remaining = grant.allocation - grant.withdrawn;
171        grant.withdrawn += uint128(remaining);

186    }
```

claim(),

```Solidity
File: contracts/ARCDVestingVault.sol

228    function claim(uint256 amount) external override nonReentrant {

       // Some code

241        if (amount == withdrawable) {
242            grant.withdrawn += uint128(withdrawable);
243        } else {
244            grant.withdrawn += uint128(amount);
245            withdrawable = amount;
246        }

       // Some code
```

_syncVotingPower(),

```Solidity
File: contracts/ARCDVestingVault.sol

    function _syncVotingPower(address who, ARCDVestingVaultStorage.Grant storage grant) internal {
        History.HistoricalBalances memory votingPower = _votingPower();

        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);

        uint256 newVotingPower = grant.allocation - grant.withdrawn;

        // get the change in voting power. voting power can only go down
        // since the sync is only called when tokens are claimed or grant revoked
>>        int256 change = int256(newVotingPower) - int256(grant.latestVotingPower);
        // we multiply by -1 to avoid underflow when casting
        votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));

        grant.latestVotingPower = newVotingPower;

        emit VoteChange(grant.delegatee, who, change);
    }
```

These incorrect downcasting will create accounting errors and this is not the right way to downcast.

### Recommended Mitigation steps,
Even though Solidity 0.8.18 is used, type casts do not throw an error.
Openzeppelin  [SafeCast](https://docs.openzeppelin.com/contracts/4.x/api/utils#SafeCast) library must be used everywhere a typecast is done.

### [L&#x2011;06]  gscApprove() does not check amount validation per Natspec comment
In ArcadeTreasury.sol, gscApprove() Natspec comment has some required validation before the amount is passed to approve by gsc. The comment states,

```Solidity
182     * @notice function for the GSC to approve tokens to be pulled from the treasury. The
183     *         amount to be approved must be less than or equal to the GSC's allowance for that specific token.
```
However, there is no such validation to amount in gscApprove().

There is 1 instance of this issue:

### Recommended Mitigation steps
```Solidity
File: contracts/ArcadeTreasury.sol

    function gscApprove(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
        if (spender == address(0)) revert T_ZeroAddress("spender");
        if (amount == 0) revert T_ZeroAmount();
+       require(amount <= gscAllowance[token], "invalid allowance amount");

        // Will underflow if amount is greater than remaining allowance
        gscAllowance[token] -= amount;

        _approve(token, spender, amount, spendThresholds[token].small);
    }
```

### [L&#x2011;07]  _spend() does not destination address could be contract address with no receive()
_spend() is used to sent the native or ERC20tokens which are to be spend to destination address. However it does not check the destination address is a contract address with receive() or fallback(). The destination address existence check is also missing for destination address if it turned out to be contract address. Recommend the contract existence check and also check it can receive the ethers or there should not be revert() while sending ethers to contract address.

There is [1](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L358-L373) instance of this issue:

### [L&#x2011;08]  Missing event in setTimelock() and setManager()
While setting and updating the state variables an event must be emitted for end users information. Events are the cheapest form of storage in blockchain. Recommend to add event and emit in setTimelock() and setManager().

There are 2 instances of this issue.
[code link](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L68-L84)

### [L&#x2011;09]  staleBlockLag should not be immutable to ever changing block formation duration
In BaseVotingVault.sol, staleBlockLag variable is made immutable and passed in constructor. It is used to set the number of blocks before which the delegation history is forgotten. The issue here is it should not be made immutable as the block duration time keeps changing, Presently Ethereum has block formation time is 12 seconds. Earlier it was between 13-15 seconds. In future upgrades, the block formation time might be further reduced as the blockchain technology is under developement and changes are getting proposed. In future if the contracts are planned to deployed on various L2 like polygon, BNB, ARB, etc. then for each blockchains the block time is 2s, 3s, etc. Therefore recommend to avoid making staleBlockLag as immutable variable.

There is [1](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L36-L52C1) instance of this issue:

```Solidity
File: contracts/BaseVotingVault.sol

36     uint256 public immutable staleBlockLag;

52    constructor(IERC20 _token, uint256 _staleBlockLag) {
53        if (address(_token) == address(0)) revert BVV_ZeroAddress("token");
54        if (_staleBlockLag >= block.number) revert BVV_UpperLimitBlock(_staleBlockLag);
55
56        token = _token;
57        staleBlockLag = _staleBlockLag;
58    }
```

### [L&#x2011;10]  address(0) can be delegated in delegate()
In ARCDVestingVault.sol, delegate() can be used to delegate the address(0) which is not a desired behavior.

There is 1 instance of this issue:

```Solidity
File: contracts/ARCDVestingVault.sol

260    function delegate(address to) external {

       // Some code

282    }
```

### Recommended Mitigation steps

```Solidity
File: contracts/ARCDVestingVault.sol

    function delegate(address to) external {
+       require(to != address(0), "can not delegate to address(0));

       // Some code

    }
```

### [L&#x2011;11]  Any tokens directly sent to ARCDVestingVault.sol will be permanently locked
In ARCDVestingVault.sol, Natspec states,

```Solidity
 * @dev There is no emergency withdrawal, any funds not sent via deposit() are unrecoverable
```

However, if the tokens are sent to contract will be permanently locked as there is no withdrawal function.

### Recommended Mitigation steps
Add recoverFunds() function with owner access in contract so that the tokens can be recovered.

### [L&#x2011;12]  Avoid setting null _merkleRoot in setMerkleRoot()
In ArcadeAirdrop.sol, setMerkleRoot() is used to set merkel root which is of users getting air dropped. It is recommeded to validate the merkle root before setting it.

There is 1 instance of this issue:

```Solidity
File: contracts/token/ArcadeAirdrop.sol

75    function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
76        rewardsRoot = _merkleRoot;
77
78        emit SetMerkleRoot(_merkleRoot);
79    }
```

### Recommended Mitigation steps
```Solidity
File: contracts/token/ArcadeAirdrop.sol

    function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
+       require(_merkleRoot != bytes32(0), "invalid merkle root");
        rewardsRoot = _merkleRoot;

        emit SetMerkleRoot(_merkleRoot);
    }
```

### [L&#x2011;13]  Use latest version of openzeppelin library for mitigating MerkleProof.sol bug
ReputationBadge.sol has used the MerkleProof.sol contract which is from openzeppelin. However MerkleProof.sol has a known bugs which must be fixed. Openzeppelin has accepted the bug and fixed the bug in its latest version.

CVE Bug reference:- https://nvd.nist.gov/vuln/detail/CVE-2023-34459
Openzeppelin reference:- https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-wprv-93r4-jj2p

In addition, openzeppelin also updates the contracts with gas optimizations and overall best security with each update.

The contracts has used the old version which is security proof and latest version has fixed lots of bugs and gas optimizations.

There is 1 instance of this issue:-
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L7C52-L7C63

### Recommended Mitigation steps
Update the openzeppelin library to latest version.

### [L&#x2011;14]  Misleading comment on upgradeable contracts
In NFTBoostVault.sol, the comments states, NFTBoostVault contract is upgradeable,

```Solidity
46    * This contract is Simple Proxy upgradeable which is the upgradeability system used for voting
47    * vaults in Council.
```
However, per the discussion with sponsor none of the smart contracts in scope are upgradeable. Therefore the comment is misleading and it is incorrect.

### Recommended Mitigation steps
Delete the comment as the smart contracts are not upgradeable.