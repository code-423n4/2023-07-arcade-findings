`delegate` function updates voting power and emits events, which can consume gas when called frequently. Specifically, gas consumption can increase due to the emission of events and the manipulation of historical voting power data.

[delegate Function:](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L260-L282)
```solidity
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

    // add voting power to the target delegatee and emit event
    votingPower.push(to, newDelegateeVotes + grant.latestVotingPower);

    // update grant delegatee info
    grant.delegatee = to;

    emit VoteChange(to, msg.sender, int256(uint256(grant.latestVotingPower)));
}
```
The emission of events adds gas because it requires the contract to store the event data on-chain. This data can be relatively large, depending on the amount of information that is being tracked. For example, the `VoteChange` event in the ARCDVestingVault contract stores the addresses of the grant holder, the old `delegatee`, the new `delegatee`, and the amount of voting power that was transferred.

The manipulation of historical voting power data also adds gas because it requires the contract to update the state of the voting power table. This table tracks the voting power of all grant holders, and it must be updated whenever a grant holder's voting power is changed. The more frequently the delegate function is called, the more often the voting power table will need to be updated, and the more gas will be consumed. 

The following gas-intensive operations can be observed:
1. The `VoteChange` event is emitted twice within the function, which can consume gas, especially when the function is called frequently.
2. The function manipulates historical voting power data through the `votingPower.push` calls, which can be costly when executed frequently.


## Recommended Mitigation Steps
Consider the usage and frequency of the `delegate` function to optimize gas usage and minimize transaction costs. Frequent calls to this function might lead to increased gas fees for users interacting with the contract.