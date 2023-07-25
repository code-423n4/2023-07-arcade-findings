# GAS OPTIMIZATION REPORT 

## [G-O1] claim function in ARCDVestingVault.sol is doing an unneccesary check

claim function is doing the next checking :

```
 function claim(uint256 amount) external override nonReentrant {
        ...
        if (amount > withdrawable) revert AVV_InsufficientBalance(withdrawable);

        // update the grant's withdrawn amount
        if (amount == withdrawable) {
            grant.withdrawn += uint128(withdrawable);
        } else {
            grant.withdrawn += uint128(amount);
            withdrawable = amount;
        } 
        ...
    }
```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L228

The function above can be rewrite like the code below and it is going to do the same logic:

```
 function claim(uint256 amount) external override nonReentrant {
        ...
        if (amount > withdrawable) revert AVV_InsufficientBalance(withdrawable);

        // update the grant's withdrawn amount
        
            withdrawable = amount;
            grant.withdrawn += uint128(withdrawable);
        ...
    }
```

the result is eviting an if stament wich cost gas.

## [G-O1] create modifier to check address 0 in ArcadeTokenDistributor.sol

the contract is performing checkins for 0 address in all functions, this can be replace for a modifier a safe gas in each call

```
modifier zeroAddress(address _address){
if (_address == address(0)) { revert}
}
```

