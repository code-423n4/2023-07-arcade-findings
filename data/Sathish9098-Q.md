
# LOW FINDINGS

##

##

## [L-1] Use safeInceaseAllowance()/safeDecreaseAllowance() instead of approve() or deprecated safeApprove() Functions

### Impact
The ``safeIncreaseAllowance()`` and ``safeDecreaseAllowance()`` functions also prevent ``front-running attacks`` by atomically updating the allowance and emitting an Approval event. This means that the allowance cannot be changed by another transaction between the time the allowance is updated and the Approval event is emitted. 

The safeApprove() function was deprecated because it was not gas-efficient and it did not prevent front-running attacks. You should use safeIncreaseAllowance() or safeDecreaseAllowance() instead of safeApprove()

### POC

```solidity
FILE: Breadcrumbs2023-07-arcade/contracts/ArcadeTreasury.sol

200:  _approve(token, spender, amount, spendThresholds[token].small);

219: _approve(token, spender, amount, spendThresholds[token].small);

238:  _approve(token, spender, amount, spendThresholds[token].medium);

257: _approve(token, spender, amount, spendThresholds[token].large);

391:   IERC20(token).approve(spender, amount);

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L391

### Recommended Mitigation
Use ``safeIncreaseAllowance()`` or ``safeDecreaseAllowance() `` instead of approve() or safeApprove()

##

## [L-2] Don’t use payable.transfer()

### Impact
The use of ``payable.transfer()`` is [heavily frowned upon](https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/) because it can lead to the locking of funds. The transfer() call requires that the recipient is either an EOA account, or is a contract that has a payable callback. For the contract case, the transfer() call only provides 2300 gas for the contract to complete its operations. This means the following cases can cause the transfer to fail:

- The contract does not have a payable callback
- The contract’s payable callback spends more than 2300 gas (which is only enough to emit something)

### POC

```solidity
FILE: 2023-07-arcade/contracts/ArcadeTreasury.sol

367: payable(destination).transfer(amount);

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L367

```solidity
FILE: 2023-07-arcade/contracts/nft/ReputationBadge.sol

171: payable(recipient).transfer(balance);

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L171

##

## [L-3] Hardcoded cooldown period may cause problem in future 

### Impact
Hardcoding a cooldown period in the contract may lead to potential problems in the future. The hardcoded value of ``SET_ALLOWANCE_COOL_DOWN`` is currently set to ``7 days``, which means the cooldown period for updating the GSC allowance is fixed and cannot be changed without redeploying the contract.

### POC

```solidity
FILE: 2023-07-arcade/contracts/ArcadeTreasury.sol

56: uint48 public constant SET_ALLOWANCE_COOL_DOWN = 7 days;

```

### Recommended Mitigation
Cooldown period can be set by the contract owner or by a governance mechanism. This allows the cooldown period to be adjusted as needed, without having to modify the contract code.

##

## [L-4] 






## [L-1] Lack of same value input control in critical setMinter() functions

###
The same address value should be avoided 

### POC

```
FILE: 2023-07-arcade/contracts/token/ArcadeToken.sol

 function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

        minter = _newMinter;
        emit MinterUpdated(minter);
    }


```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132-L137

### Recommended Mitigation

Add same value input control 

```
require(_newMinter!=minter, "Same address value ")
```





