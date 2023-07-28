0. Gas optimizations:

https://github.com/code-423n4/2023-07-arcade/blob/34b2821a40d4e73f70056cbd675bba079e66f05e/contracts/NFTBoostVault.sol#L342-L349


    function updateVotingPower(address[] calldata userAddresses) public override {
        if (userAddresses.length > 50) revert NBV_ArrayTooManyElements();
        uint256 userAddressesLength = userAddresses.length;
        for (uint256 i; i < userAddressesLength; ) {
            NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[userAddresses[i]];
            _syncVotingPower(userAddresses[i], registration);
            
            unchecked {
            	++i;
            }
        }
    }

