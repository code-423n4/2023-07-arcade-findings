### If statements
These if statments can be changed to ternary operator to reduce deployment size and deployment costs

1. [Line 47](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/CoreVoting.sol#L47-L51) in CoreVoting contract
2. [Line 365](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L365-L370) in ArcadeTreasury contract
3. [Line 589](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L589-L594) in NFTBoostVault contract
4. [Line 95](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/vaults/GSCVault.sol#L95-L99) and [Line 159](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/vaults/GSCVault.sol#L159-L167) in GSCVault contract

Here is an example:
```solidity
return num > 0 ? 1 : 0;
```

### Check for `votingPower`
 
In CoreVoting contract you get total vote power by calling `queryVotePower` function and you cache sum of all in [`votingPower`](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/CoreVoting.sol#L225) variable but at the end of for loop you don't check if `votingPower` is zero, you keep executing the code for `0` power.
You can revert the tx if `votingPower` is zero and caller doesn't lose more gas (the remainig gas will be refunded to caller if tx reverts)

### `proposal` function update the state before validation
In [CoreVoting::proposal](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/CoreVoting.sol#L172-L195) you validate the vote power of caller after you added the new proposal to state, so if callers doesn't have enough vote power function will revert and they lose more gas.

You can update the state after validation to reduce gas cost in case of failure
```solidity
        uint256 votingPower = vote(votingVaults, extraVaultData, proposalCount, ballot);

        uint256 minPower = quorum <= minProposalPower ? quorum : minProposalPower;

        if (!isAuthorized(msg.sender)) {
            require(votingPower >= minPower, "insufficient voting power");
        }

        proposals[proposalCount] = Proposal(
            proposalHash,
            uint128(block.number - 1),
            uint128(block.number + lockDuration),
            uint128(block.number + lockDuration + extraVoteTime),
            uint128(quorum),
            proposals[proposalCount].votingPower,
            uint128(lastCall)
        );
```

### Do not use `assert`

In GSCVault contract at [line 61](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/vaults/GSCVault.sol#L61) you use `assert` for validating the users input, `assert` consume all of the available gas if it fails.
You should use `revert` insted, because the remaining gas will be refunded to caller.