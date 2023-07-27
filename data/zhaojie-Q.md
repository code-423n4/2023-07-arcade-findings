
## Dependent functions (outside the scope) that can use more efficient algorithms.

GSCVault.proveMembership

```
   function proveMembership(
        address[] calldata votingVaults,
        bytes[] calldata extraData
    ) external {
        // Check for call validity
        assert(votingVaults.length > 0);
        // We loop through the voting vaults to check they are authorized
        // We check all up front to prevent any reentrancy or weird side effects
        //@audit  <----> for loops can be merged 
        for (uint256 i = 0; i < votingVaults.length; i++) {
            // No repeated vaults in the list
            for (uint256 j = i + 1; j < votingVaults.length; j++) {
                require(votingVaults[i] != votingVaults[j], "duplicate vault");
            }
            // Call the mapping the core voting contract to check that
            // the provided address is in fact approved.
            // Note - Post Berlin hardfork this repeated access is quite cheap.
            bool vaultStatus = coreVoting.approvedVaults(votingVaults[i]);
            require(vaultStatus, "Voting vault not approved");
        }
        // Now we tally the caller's voting power
        uint256 totalVotes = 0;
        // Parse through the list of vaults
        for (uint256 i = 0; i < votingVaults.length; i++) {
            // Call the vault to check last block's voting power
            // Last block to ensure there's no flash loan or other
            // intra contract interaction
            uint256 votes =
                IVotingVault(votingVaults[i]).queryVotePower(
                    msg.sender,
                    block.number - 1,
                    extraData[i]
                );
            // Add up the votes
            totalVotes += votes;
        }
        // Require that the caller has proven that they have enough votes
        //@audit  <----> can put it in the for loop
        require(totalVotes >= votingPowerBound, "Not enough votes");
        // if the caller has already provedMembership, update their votingPower without
        // resetting their idle duration.
        if (members[msg.sender].joined != 0) {
            members[msg.sender].vaults = votingVaults;
        } else {
            members[msg.sender] = Member(votingVaults, block.timestamp);
        }
        // Emit the event tracking this
        emit MembershipProved(msg.sender, block.timestamp);
    }

```

The modified code is as follows:

```
function proveMembership11(
        address[] calldata votingVaults,
        bytes[] calldata extraData
    ) external {
        // Check for call validity
        assert(votingVaults.length > 0);
        // We loop through the voting vaults to check they are authorized
        // We check all up front to prevent any reentrancy or weird side effects
        uint256 totalVotes = 0;

        for (uint256 i = 0; i < votingVaults.length; i++) {
            // No repeated vaults in the list
            for (uint256 j = i + 1; j < votingVaults.length; j++) {
                require(votingVaults[i] != votingVaults[j], "duplicate vault");
            }
            // Call the mapping the core voting contract to check that
            // the provided address is in fact approved.
            // Note - Post Berlin hardfork this repeated access is quite cheap.
            bool vaultStatus = coreVoting.approvedVaults(votingVaults[i]);
            require(vaultStatus, "Voting vault not approved");

             // Call the vault to check last block's voting power
            // Last block to ensure there's no flash loan or other
            // intra contract interaction
            uint256 votes =
                IVotingVault(votingVaults[i]).queryVotePower(
                    msg.sender,
                    block.number - 1,
                    extraData[i]
                );
            // Add up the votes
            totalVotes += votes;

            require(totalVotes >= votingPowerBound, "Not enough votes");
        }
       
        // Require that the caller has proven that they have enough votes
        // if the caller has already provedMembership, update their votingPower without
        // resetting their idle duration.
        if (members[msg.sender].joined != 0) {
            members[msg.sender].vaults = votingVaults;
        } else {
            members[msg.sender] = Member(votingVaults, block.timestamp);
        }
        // Emit the event tracking this
        emit MembershipProved(msg.sender, block.timestamp);
    }
```

Although this contracts are not in scope, there is still a need for improvement.