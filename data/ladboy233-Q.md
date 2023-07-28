### Issue 1:

merkle leaf value should not be 64 bytes before hashing in MerkleReward Distribution

###  Issue 2:

_syncVotingPower can revert in underflow in NFTBoostVault.sol and ARCDVestingVault.sol

-------

# merkle leaf value should not be 64 bytes before hashing in MerkleReward Distribution

# line of code:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/external/council/libraries/MerkleRewards.sol#L85

## Impact

merkle leaf value should not be 64 bytes before hashing

## Proof of Concept

In merkle reward distribution code, we have to follow function:

```solidity
   function _validateWithdraw(
        uint256 amount,
        uint256 totalGrant,
        bytes32[] memory merkleProof
    ) internal {
        // Hash the user plus the total grant amount
        bytes32 leafHash = keccak256(abi.encodePacked(msg.sender, totalGrant));

        // Verify the proof for this leaf
        require(
            MerkleProof.verify(merkleProof, rewardsRoot, leafHash),
            "Invalid Proof"
        );
        // Check that this claim won't give them more than the total grant then
        // increase the stored claim amount
        require(claimed[msg.sender] + amount <= totalGrant, "Claimed too much");
        claimed[msg.sender] += amount;
    }
```

note:

```solidity
// Hash the user plus the total grant amount
bytes32 leafHash = keccak256(abi.encodePacked(msg.sender, totalGrant));

// Verify the proof for this leaf
require(
	MerkleProof.verify(merkleProof, rewardsRoot, leafHash),
	"Invalid Proof"
);
```

we cancat the address msg.sender and totalGrant as merkle tree, which is 64 bytes exactly

In Openzepplein library, we have the following comments

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/02ea01765a9964541dd9cdcffc4a7f8b403c2ff6/contracts/utils/cryptography/MerkleProof.sol#L13

```solidity
 * WARNING: You should avoid using leaf values that are 64 bytes long prior to
 * hashing, or use a hash function other than keccak256 for hashing leaves.
 * This is because the concatenation of a sorted pair of internal nodes in
 * the merkle tree could be reinterpreted as a leaf value.
```

The merkle tree intermediary nodes are just concatenated and hashed during verification, and since they are also of size bytes32/uint256 this type of collision is possible because the leaf of the tree is of the same form in this case (address, uint256) as the internal nodes.

then if the attacker control two msg.sender address + the totalGrant, he can claim the reward for free

## Tools Used

Manual Review

## Recommended Mitigation Steps

Add one bytes to the hashed leaf

--------

# _syncVotingPower can revert in underflow in NFTBoostVault.sol and ARCDVestingVault.sol

## line of code:

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L593

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L352

## Impact

_syncVotingPower can revert in underflow in NFTBoostVault.sol

## Proof of Concept

note this function:

```solidity
   function _syncVotingPower(address who, NFTBoostVaultStorage.Registration storage registration) internal {
        History.HistoricalBalances memory votingPower = _votingPower();
        uint256 delegateeVotes = votingPower.loadTop(registration.delegatee);

        uint256 newVotingPower = _currentVotingPower(registration);
        // get the change in voting power. Negative if the voting power is reduced

        int256 change = int256(newVotingPower) - int256(uint256(registration.latestVotingPower));

        // do nothing if there is no change
        if (change == 0) return;
        if (change > 0) {
            votingPower.push(registration.delegatee, delegateeVotes + uint256(change));
        } else {
            // if the change is negative, we multiply by -1 to avoid underflow when casting
            votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));
        }

        registration.latestVotingPower = uint128(newVotingPower);

        emit VoteChange(who, registration.delegatee, change);
    }

```

We need to pay special attention to this line of code:

```solidity
  votingPower.push(registration.delegatee, delegateeVotes - uint256(change * -1));
```

it is possible that delegateeVotes - changes can revert in underflow

this only happens when we try to reduce the voting power

In _withdrawNFT if a NFT with high multipler is removed in NFTBoostVault.sol and cause the [voting power to decrease](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L569)

```solidity
// update the delegatee's voting power based on multiplier removal
_syncVotingPower(msg.sender, registration);
```

Or in ARCDVestingVault.sol when a admin try to [revoke a user's grant](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L175)

```solidity
// update the grant's withdrawn amount
if (amount == withdrawable) {
	grant.withdrawn += uint128(withdrawable);
} else {
	grant.withdrawn += uint128(amount);
	withdrawable = amount;
}

// update the user's voting power
_syncVotingPower(msg.sender, grant);
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Before calling delegateeVotes - uint256(change * -1), make sure delegateeVotes > uint256(change * -1), otherwise just set to 0




