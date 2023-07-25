L-01: _spend() blockExpenditure Limit distinguishes token more reasonably 

Currently `ArcadeTreasury.sol` methods `_spend()```_approve``, limit do not distinguish `token``, resulting in the possibility of affecting each other, it is suggested that distinguishing `token`` is more reasonable.

```solidity
    function _spend(address token, uint256 amount, address destination, uint256 limit) internal {
        // check that after processing this we will not have spent more than the block limit
-       uint256 spentThisBlock = blockExpenditure[block.number];
+       uint256 spentThisBlock = blockExpenditure[token][block.number];
        if (amount + spentThisBlock > limit) revert T_BlockSpendLimit();
-       blockExpenditure[block.number] = amount + spentThisBlock;
+       blockExpenditure[token][block.number] = amount + spentThisBlock;

....


    }

    function _approve(address token, address spender, uint256 amount, uint256 limit) internal {
        // check that after processing this we will not have spent more than the block limit
-       uint256 spentThisBlock = blockExpenditure[block.number];
+       uint256 spentThisBlock = blockExpenditure[token][block.number];
        if (amount + spentThisBlock > limit) revert T_BlockSpendLimit();
-       blockExpenditure[block.number] = amount + spentThisBlock;
+       blockExpenditure[token][block.number] = amount + spentThisBlock;

        // approve tokens
        IERC20(token).approve(spender, amount);

        emit TreasuryApproval(token, spender, amount);
    }    
```    


L-02: NFTBoostVault.delegate()  Lack of checking `delegatee` != 0
```solidity
    function delegate(address to) external override {
        if (to == address(0)) revert NBV_ZeroAddress("to");
        

        NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

        // If to address is already the delegate, don't send the tx
        if (to == registration.delegatee) revert NBV_AlreadyDelegated();
+       if (registration.delegatee == address(0)) revert NBV_NoRegistration();

        History.HistoricalBalances memory votingPower = _votingPower();
        uint256 oldDelegateeVotes = votingPower.loadTop(registration.delegatee);
```    


L-03: updateNft() Lack of check the newTokenAddress is legal

```solidity
    function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {
        if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);

        if (IERC1155(newTokenAddress).balanceOf(msg.sender, newTokenId) == 0) revert NBV_DoesNotOwn();

        NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

.....

        // set the new ERC1155 values in the registration and lock the new ERC1155
        registration.tokenAddress = newTokenAddress;
        registration.tokenId = newTokenId;

+       uint128 multiplier = getMultiplier(newTokenAddress, newTokenId);
+       if (multiplier == 0) revert NBV_NoMultiplierSet();

        _lockNft(msg.sender, newTokenAddress, newTokenId, 1);

        // update the delegatee's voting power based on new ERC1155 nft's multiplier
        _syncVotingPower(msg.sender, registration);
    }
```    