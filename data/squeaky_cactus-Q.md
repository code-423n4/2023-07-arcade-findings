# QA Report

| Id                                                                        | Issue                                                      |
|---------------------------------------------------------------------------|------------------------------------------------------------|
| [L-1](#l-1-input---validate-tokenid-uniqueness)                           | Input - Validate tokenId uniqueness                        |
| [L-2](#l-2-input---new-minter-can-be-same-as-the-old-minter)              | Input - Input - New minter can be same as the old minter   |
| [L-3](#l-3-input---timelock-can-be-same-as-the-timelock)                  | Input - Input - Timelock can be same as the timelock       |
| [L-4](#l-4-input---manager-can-be-same-as-the-manager)                    | Input - Input - Manager can be same as the manager         |
| [NC-1](#nc-1-natspec---misleading-function-application)                   | NatSpec - Misleading function application                  |
| [NC-2](#nc-2-natspec---imprecise-time-delay-requirement)                  | NatSpec - Imprecise time delay requirement                 |
| [NC-3](#nc-3-natspec---missing-apostrophe)                                | NatSpec - Missing apostrophe                               |
| [NC-4](#nc-4-natspec---external-parameter-specificity)                    | NatSpec - external parameter specificity                   |
| [NC-5](#nc-5-code-style---parameter-prefix-and-suffix)                    | Code Style - Parameter prefix and suffix                   |
| [NC-6](#nc-6-inline-comment---invalid-state-assertion)                    | Inline comment - Invalid state assertion                   |
| [NC-7](#nc-7-event---no-emitted-event-on-state-change)                    | Event - No emitted event on state change                   |
| [NC-8](#nc-8-natspec---incorrect-statement-on-supporting-upgradeability)  | NatSpec - Incorrect statement on supporting upgradeability |



## Low Risk 

### L-1 Input - Validate tokenId uniqueness
The function `ReputationBadge::publishRoots()` iterates over a given array of `ClaimData`, storing by `tokenId`
(part of the `ClaimData` struct).

Within the `external` function no validation is performed on uniqueness of the `tokenId` within the given data set, 
resulting in permitting multiple claim roots for a single claim Id, each being overriden, leaving only the latest element
of the given `ClaimData` array.

In contrast, there is validation for the `claimExpiration` element of `ClaimData`, suggesting validation is desired.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L135C1-L156C6

#### Recommended Mitigation
Update the NatSpec to be explicit about input expectation.
(A uniqueness check in the function is an option, but that will increase gas by a quadratic relationship to array size).

```
-     * @param _claimData        The claim data to update.
+     * @param _claimData        The claim data to update, where if the tokenId is not unique, each entry is overridden by the later one.
```



### L-2 Input - New minter can be same as the old minter
In `ArcadeToken::setMinter` the new minter can be equal to the existing minter. 
A potential waste of gas and event noise.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L127-L138
```
    /**
     * @notice Function to change the minter address. Can only be called by the minter.
     *
     * @param _newMinter            The address of the new minter.
     */
    function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

        minter = _newMinter;
        emit MinterUpdated(minter);
    }
```

#### Recommended Mitigation
Require the new minter is not the current minter and create a custom error.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L133
```
+    if (_newMinter == minter) revert AT_MinterSame(_newMinter);
```



### L-3 Input - Timelock can be same as the timelock
In `BaseVotingVault::setTimelock` the new timelock can be equal to the existing timelock.
A potential waste of gas.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L68-L72
```
    function setTimelock(address timelock_) external onlyTimelock {
        if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");

        Storage.set(Storage.addressPtr("timelock"), timelock_);
    }
```

#### Recommended Mitigation
Require the new timelock is not the current timelock and create a custom error.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L68-L72
```
+      if (timelock_ == _timelock()) revert BVV_SameAddress("timelock");
```



### L-4 Input - Manager can be same as the manager
In `BaseVotingVault::setManager` the new manager can be equal to the existing manager.
A potential waste of gas.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L80-L84
```
    function setManager(address manager_) external onlyTimelock {
        if (manager_ == address(0)) revert BVV_ZeroAddress("manager");

        Storage.set(Storage.addressPtr("manager"), manager_);
    }
```

#### Recommended Mitigation
Require the new manager is not the current manager and create a custom error.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L80-L84
```
+       if (manager_ == _manager()) revert BVV_SameAddress("manager");
```






## Non-Critical

### NC-1 NatSpec - Misleading function application
The NatSpec for function `BadgeDescriptor::tokenURI` misleadingly qualifies application to 
ERC721 tokenIds, while it is being used for ERC1155 tokenIds.

The implementation of the function is abstract from any interface, so can be applied to any
tokenId.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/BadgeDescriptor.sol#L42
```
     * @notice Getter of specific URI for an ERC721 token ID.
```

#### Recommended Mitigation
Drop the qualifier.
```
-     * @notice Getter of specific URI for an ERC721 token ID.
+     * @notice Getter of specific URI for a token ID.
```



### NC-2 NatSpec - Imprecise time delay requirement
The NatSpec for `ArcadeToken` includes the phrase `the minter must wait at least 1 year before calling it again.`.
Implementation uses `365 days` for the constant of one year, however that does not hold true on years with leap days.

"Take care if you perform calendar calculations using these units, because not every year equals 365 days"
https://docs.soliditylang.org/en/v0.8.18/units-and-global-variables.html#time-units

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L71-L72
```
 * An inflationary cap of 2% per year is enforced on each mint. Every time the mint function
 * is called, the minter must wait at least 1 year before calling it again.
```

#### Recommended Mitigation
Change the comment to Use 365 days.

```
- * is called, the minter must wait at least 1 year before calling it again.
+ * is called, the minter must wait at least 365 days before calling it again.
```



### NC-3 NatSpec - Missing apostrophe
Missing the apostrophe in that's.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L101
```
    /// @dev An event thats emitted when the minter address is changed
```

#### Recommended Mitigation
Correct the spelling.

```
-    /// @dev An event thats emitted when the minter address is changed
+    /// @dev An event that's emitted when the minter address is changed
```



### NC-4 NatSpec - external parameter specificity
The parameter NatSpec for `ArcadeToken::mint` is lacking specificity on the revert conditions.

More precise documentation for `public` and `external` functions helps when calling taking a black box approach
e.g. invoking functions on verified contract in Etherscan.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L139-L148
```
    /**
     * @notice Mint Arcade tokens. Can only be called by the minter.
     *
     * @param _to                 The address to mint tokens to.
     * @param _amount             The amount of tokens to mint.
     */
    function mint(address _to, uint256 _amount) external onlyMinter {
        if (block.timestamp < mintingAllowedAfter) revert AT_MintingNotStarted(mintingAllowedAfter, block.timestamp);
        if (_to == address(0)) revert AT_ZeroAddress("to");
        if (_amount == 0) revert AT_ZeroMintAmount();
```

#### Recommended Mitigation
Update the NatSpec.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L142-L143
```
-     * @param _to                 The address to mint tokens to.
-     * @param _amount             The amount of tokens to mint.

+     * @param _to                 The address to mint tokens to, not address zero.
+     * @param _amount             The amount of tokens to mint, greater than zero.

```



### NC-5 Code Style - Parameter prefix and suffix
In `ARCDVestingVault::constructor` has four parameters, using underscores as a prefix on two of them and as an underscore 
on the other two. This usage is inconsistent with all other functions in `ARCDVestingVault` whose parameters use no underscore, while elsewhere
in the codebase, predominately underscores are used as prefix.

While [SolidityLang](https://docs.soliditylang.org/en/latest/style-guide.html) has no convention on parameter styling,
there is a strong argument for keeping consistency across the codebase as a whole, or at least at the file level.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L57-L66
```
    /**
     * @notice Deploys a new vesting vault, setting relevant immutable variables
     *         and granting management power to a defined address.
     *
     * @param _token              The ERC20 token to grant.
     * @param _stale              Stale block used for voting power calculations
     * @param manager_            The address of the manager.
     * @param timelock_           The address of the timelock.
     */
    constructor(IERC20 _token, uint256 _stale, address manager_, address timelock_) BaseVotingVault(_token, _stale) {
```

#### Recommended Mitigation
Keep consistency within the file, drop underscore usage.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L142-L143
```
-     * @param _token              The ERC20 token to grant.
-     * @param _stale              Stale block used for voting power calculations
-     * @param manager_            The address of the manager.
-     * @param timelock_           The address of the timelock.
-    constructor(IERC20 _token, uint256 _stale, address manager_, address timelock_) BaseVotingVault(_token, _stale) {

+     * @param token              The ERC20 token to grant.
+     * @param stale              Stale block used for voting power calculations
+     * @param manager            The address of the manager.
+     * @param timelock           The address of the timelock.
-    constructor(IERC20 token, uint256 stale, address manager, address timelock) BaseVotingVault(token, stale) {
```



### NC-6 Inline comment - Invalid state assertion
In `ARCDVestingVault::delegate()` there is an inline comment asserting state that cannot come to pass due to a revert guard.

`to == grant.delegatee` can never be true, due to the guard line above ` if (to == grant.delegatee) revert AVV_AlreadyDelegated();`.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L260-L273
```
    function delegate(address to) external {
        ARCDVestingVaultStorage.Grant storage grant = _grants()[msg.sender];
        if (to == grant.delegatee) revert AVV_AlreadyDelegated();

        History.HistoricalBalances memory votingPower = _votingPower();
        uint256 oldDelegateeVotes = votingPower.loadTop(grant.delegatee);

        // Remove old delegatee's voting power and emit event
        votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);
        emit VoteChange(grant.delegatee, msg.sender, -1 * int256(grant.latestVotingPower));

        // Note - It is important that this is loaded here and not before the previous state change because if
        // to == grant.delegatee and re-delegation was allowed we could be working with out of date state.
        uint256 newDelegateeVotes = votingPower.loadTop(to);
```


#### Recommended Mitigation
Delete the redundant comment.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L271-L272
```
-        // Note - It is important that this is loaded here and not before the previous state change because if
-        // to == grant.delegatee and re-delegation was allowed we could be working with out of date state.
```



### NC-7 Event - No emitted event on state change

Instances: 2

In the `external` function `BaseVotingVault::setTimelock()` and `BaseVotingVault::setManager()`, contract state is 
mutated, but no event is emitted.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L68-L72
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L80-L84

```  
  function setTimelock(address timelock_) external onlyTimelock {
        if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");

        Storage.set(Storage.addressPtr("timelock"), timelock_);
    }

...

    function setManager(address manager_) external onlyTimelock {
        if (manager_ == address(0)) revert BVV_ZeroAddress("manager");

        Storage.set(Storage.addressPtr("manager"), manager_);
    }
```  

#### Recommended Mitigation
Create an events for each mutation and emit them appropriately.

```
+       event TimelockChange(address indexed from, address indexed to);

+       event ManagerChange(address indexed from, address indexed to);

    function setTimelock(address timelock_) external onlyTimelock {
        if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");
        Storage.set(Storage.addressPtr("timelock"), timelock_);
+       emit TimelockChange (_timelock(), timelock_ );
    }

...

    function setManager(address manager_) external onlyTimelock {
        if (manager_ == address(0)) revert BVV_ZeroAddress("manager");
        Storage.set(Storage.addressPtr("manager"), manager_);
+       emit ManagerChange( _manager(), manager_ );
    }
```



### NC-8 NatSpec - Incorrect statement on supporting upgradeability
The contract NatSpec for `NFTBoostVault` states it supports simple upgradability, while the contract does not.

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L46-L47
```
  * This contract is Simple Proxy upgradeable which is the upgradeability system used for voting
  * vaults in Council.
```

#### Recommended Mitigation
Remove the offending lines from the contract NatSpec.

```
-  * This contract is Simple Proxy upgradeable which is the upgradeability system used for voting
-  * vaults in Council.
```
