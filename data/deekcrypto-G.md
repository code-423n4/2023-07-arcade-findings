# ARCDVestingVault.sol

## 1
Reduce gas cost by 2 by using the ternary operator instead of the if statement.

Lines 105-107:
```
    if (startTime == 0) {
       startTime = uint128(block.number);
    }
```
New line:
``` 
   startTime = startTime == 0 ? uint128(block.number) : startTime;
```

## 2
Unnecessary new variable declaration `uint128 newVotingPower`. The variable `amount` is not modified after its declaration, so using the existing variable instead is better. Reduces gas cost by 45.

Lines 128-148:
```
    uint128 newVotingPower = amount;

    // set the new grant
    grant.allocation = amount;
    grant.cliffAmount = cliffAmount;
    grant.withdrawn = 0;
    grant.created = startTime;
    grant.expiration = expiration;
    grant.cliff = cliff;
    grant.latestVotingPower = newVotingPower;
    grant.delegatee = delegatee;

    // update the amount of unassigned tokens
    unassigned.data -= amount;

    // update the delegatee's voting power
    History.HistoricalBalances memory votingPower = _votingPower();
    uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
    votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);

    emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
```

Suggested change:
```
    // set the new grant
    grant.allocation = amount;
    grant.cliffAmount = cliffAmount;
    grant.withdrawn = 0;
    grant.created = startTime;
    grant.expiration = expiration;
    grant.cliff = cliff;
    grant.latestVotingPower = amount;
    grant.delegatee = delegatee;

    // update the amount of unassigned tokens
    unassigned.data -= amount;

    // update the delegatee's voting power
    History.HistoricalBalances memory votingPower = _votingPower();
    uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
    votingPower.push(grant.delegatee, delegateeVotes + amount);

    emit VoteChange(grant.delegatee, who, int256(uint256(amount)));
```

## 3
The if-else statement can be simplified. Reduces gas cost by 34.

Line 138 ensures that `amount > withdrawable`:
```
    if (amount > withdrawable) revert AVV_InsufficientBalance(withdrawable);
```

Lines 241-246
```
    if (amount == withdrawable) {
        grant.withdrawn += uint128(withdrawable); // Same as uint128(amount)
    } else {
        grant.withdrawn += uint128(amount);
        withdrawable = amount;
    }

    // update the user's voting power
    _syncVotingPower(msg.sender, grant);

    // transfer the available amount
    token.safeTransfer(msg.sender, withdrawable);
```
        
Suggested change:
```
    grant.withdrawn += uint128(amount);

    // update the user's voting power
    _syncVotingPower(msg.sender, grant);

    // transfer the available amount
    token.safeTransfer(msg.sender, amount);
```


## 4
Inefficient usage of `int256` when `uint256` can be utilized.

Lines 348-352:
```
    // get the change in voting power. voting power can only go down
    // since the sync is only called when tokens are claimed or grant revoked
    int256 change = int256(newVotingPower) - int256(grant.latestVotingPower);
    // we multiply by -1 to avoid underflow when casting
    votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));

    grant.latestVotingPower = newVotingPower;

    emit VoteChange(grant.delegatee, who, change);
```

As mentioned, the voting power can only go down. Therefore, it is safe to assume that the value of `change` is always negative value, so `uint256` can be used instead used to optimize gas usage.

Suggested change:
```
    // get the change in voting power. voting power can only go down
    // since the sync is only called when tokens are claimed or grant revoked
    uint256 change = grant.latestVotingPower - newVotingPower;  // Save gas by removing two typecasts here
    // we multiply by -1 to avoid underflow when casting
    votingPower.push(grant.delegatee, delegateeVotes - change); // Save gas by removing typecast and multiplication here

    grant.latestVotingPower = newVotingPower;

    emit VoteChange(grant.delegatee, who, int256(change) * -1); // Typecast here once instead
```