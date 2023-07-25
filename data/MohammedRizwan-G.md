### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | Catch amount argument in gscSpend() function | 8 |
| [G&#x2011;02] | Use openzeppelin reentrancy guard instead of creating one | All contracts |
| [G&#x2011;03] | Set the arcadeToken address in constructor  as it can only set once | 1 |

### [G&#x2011;01]  Catch amount argument in gscSpend() function
In gscSpend(),amount argument can be catched because caching will replace each Gwarmaccess (100 gas) with a much cheaper stack read. 
There are [8](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L108-L258) instances of this issue:

### Recommended Mitigation steps

For example:

```Solidity
File: contracts/ArcadeTreasury.sol

    function gscSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
+       uint256 _amount = amount;

        if (destination == address(0)) revert T_ZeroAddress("destination");
-       if (amount == 0) revert T_ZeroAmount();
+       if (_amount == 0) revert T_ZeroAmount();

+       uint256 _amount = amount;

        // Will underflow if amount is greater than remaining allowance
-       gscAllowance[token] -= amount;
+       gscAllowance[token] -= _amount;


-        _spend(token, amount, destination, spendThresholds[token].small);
+        _spend(token, _amount, destination, spendThresholds[token].small);
    }
```

### [G&#x2011;02]   Use openzeppelin reentrancy guard instead of creating one
In ARCDVestingVault.sol, HashedStorageReentrancyBlock.sol is imported which is nothing but reentrancy guard however the contracts has used openzeppelin library therefore openzeppelin reentrany guard can be easily used instead of creating one which will extra bytes causing more deployment cost.

### Recommended Mitigation steps
Use openzeppelin reentrant modifier to protect against re-entrancy.

### [G&#x2011;03]  Set the arcadeToken address in constructor
In ArcadeTokenDistributor.sol, setToken() is called by owner to set the arcadeToken address for its distribution to various stake holders. Per the Natspec the function can only be called once,

```Solidity
169     * @notice Sets the Arcade Token contract address, to be used in token distribution. The token
170     *         contract address can only be set once.
```
It means it can not be reinitialized, however for every contract there is one constructor and that constructor can be used to set the arcadeToken address during the deployment itself. This will save these extra bytes causing a different function i.e setToken(). This will save considerable amount of gas.

There is 1 instance of this issue:

```Solidity
File: contracts/token/ArcadeTokenDistributor.sol

    function setToken(IArcadeToken _arcadeToken) external onlyOwner {
        if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");
        if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();

        arcadeToken = _arcadeToken;
    }
```
### Recommended Mitigation steps
Use constructor for setting arcadeToken address.

Note: There is one High risk issue submitted which will enforce the use of constructor in ArcadeTokenDistributor.sol if it is approved, based on that this gas saving recommendation is made.
