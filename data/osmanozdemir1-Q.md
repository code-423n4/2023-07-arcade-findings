### Summary

* \[L-01\] Token spending threshold might be the same and not ascend from small to large in `ArcadeTreasury.sol::setThreshold()`
    
* \[L-02\] Two-step role change should be implemented for the minter role in `ArcadeToken.sol`
    
* \[NC-01\] Developer notes are redundant in `ARCDVestingVault.sol` and `NFTBoostVault.sol`
    
* \[NC-02\] NatSpec @notice gives wrong information in `ARCDVestingVault.sol::claimable()`
    
* \[NC-03\] The claiming period for airdrops will be relatively short in case of a root change, due to the immutable expiration time
    

---

### \[L-01\] Token spending threshold might be the same and not ascend from small to large in `ArcadeTreasury.sol::setThreshold()`

According to the protocol specification, every token has spending thresholds for small, medium and large amounts, and these thresholds should be in ascending order.

The `setThreshold()` function checks if the large threshold is greater than the medium threshold, and if the medium threshold is greater than the small threshold. It doesn't check if those thresholds are equal, and it doesn't revert when it should be reverting.

[https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L276](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L276)

```solidity
File: ArcadeTreasury.sol 
     if (thresholds.large < thresholds.medium || thresholds.medium < thresholds.small) {
         revert T_ThresholdsNotAscending();
     }   
```

`thresholds.large == thresholds.medium == thresholds.small` is possible.

---

### \[L-02\] Two-step role change should be implemented for the minter role in `ArcadeToken.sol`

Important role changes should follow a two-step role change to prevent losing the roles in case of incorrect input.

The minter role can only be changed by the current minter in the `ArcadeToken.sol` contract and it follows a one-step change. In case of a wrong input, there is no way to get the minter role back and new tokens won't be minted ever.

[https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132-L137](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132C1-L137C6)

```solidity
    function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

        minter = _newMinter;
        emit MinterUpdated(minter);
    }
```

I would recommend using a two-step role change by implementing a `pendingMinter` variable and the new minters should claim the minter role themselves.

---

### \[NC-01\] Developer notes are redundant in `ARCDVestingVault.sol` and `NFTBoostVault.sol`

[https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L271-L272](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L271C1-L272C107)

[https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L197C2-L198C113](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L197C2-L198C113)

```solidity
    // Note - It is important that this is loaded here and not before the previous state change because if
    // to == grant.delegatee and re-delegation was allowed we could be working with out of date state.
```

The comments above are redundant and causes confusion since `to == grant.delegatee` and `to == registration.delegatee` is impossible as the checks have been made functions revert in this case.

The cheks are [here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L262) and [here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L188).

---

### \[NC-02\] NatSpec @notice gives wrong information in `ARCDVestingVault.sol::claimable()`

[https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L286-L295](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L286C1-L295C6)

NatSpec notice in this function is: "*Returns the claimable amount for a given* ***grant****.*"

It should be: "*Returns the claimable amount for a given* ***address****.*"

---

### \[NC-03\] The claiming period for airdrops will be relatively short in case of a root change, due to the immutable expiration time

[https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/libraries/ArcadeMerkleRewards.sol#L32](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/libraries/ArcadeMerkleRewards.sol#L32)

The owner of the `ArcadeAirdrop.sol` contract can change the `rewardsRoot` that validates the airdrop claim. There might be different reasons for changing the root but the expiration time for claims is immutable. Therefore, changing the rewards root will squeeze all the claims for all the users in a short amount of time relative to the initial claim period.

I would recommend considering changing the `expiration` state variable, not making it immutable, and giving the owner the ability to set a new expiration time if they change the `rewardsRoot` in the `setMerkleRoot()` function.