## 1. INPUT VALIDATION IS PERFORMED AFTER THE GAS CONSUMPTION FOR STORAGE ACCESS

The `NFTBoosVault.getMultiplier` function is used to get the `token multiplier` of the corresponding `ERC1155` NFT token. Here the input validation is done after the `SLOAD` and `SSTORE` operations are performed. Hence if the transaction returns during the input validation phase the gas spent on above operations will be wasted. Hence it is recommended to perform the input validation at the beginning of the function as follows:

    function getMultiplier(address tokenAddress, uint128 tokenId) public view override returns (uint128) {

        // if a user does not specify a ERC1155 nft, their multiplier is set to 1
        if (tokenAddress == address(0) || tokenId == 0) {
            return 1e3;
        }

        NFTBoostVaultStorage.AddressUintUint storage multiplierData = _getMultipliers()[tokenAddress][tokenId];

        return multiplierData.multiplier;
    } 

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L419-L424

## 2. NO NEED TO INITIALIZE THE `uint256` TO DEFAULT VALUE

The `NFTBoostVault._registerAndDelegate` function initializes the `registration.withdrawn` value to `0` during `registration` for a specific user. But this initialization is not required since by default the value is set to `0`. Hence this will save gas for an extra `SSTORE` operation.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L499
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L345

## 3. ARITHMETIC OPERATIONS CAN BE `unchecked` DUE TO PREVIOUS CONDITIONAL CHECKS

In the `NFTBoostVault.withdraw` function the following arithmetic operation can be `unchecked` due to previous conditional checks performed.

        balance.data -= amount; 

The previous conditional check is as follows and it ensures the above operation can not underflow:

        if (balance.data < amount) revert NBV_InsufficientBalance(); 


Hence the arithmetic opreation can be modified with the `unchecked` as follows:

    unchecked{
            balance.data -= amount;     
    }

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L232
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L239
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L141

## 4. USE `delete` KEYWORD INSTEAD OF RESETTING THE STORAGE VARIABLE TO DEFAULT VALUE

        registration.tokenAddress = address(0);
        registration.tokenId = 0;

The `NFTBoostVault._withdrawNft` function, is used to withdraw the `ERC1155 NFT` by the user. The corresponding registration parameters of the NFT are reset to default value after the `safeTransferFrom` call. But instead of resetting the storage variables to the default value, the `delete` can be used in its place to get a gas refund.

Hence the above mentioned lines of code can be modified as follows:

        delete registration.tokenAddress;
        delete registration.tokenId;

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L565-L566
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L251-L254
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L178-L185

## 5. REDUNDANT CONDITIONAL CHECKS CAN BE OMITTED TO SAVE GAS

In the `ARCDVestingVault.claim` function, the following conditional checks is performed to make sure that the `grant.cliff` is reached currently and if not the transaction will revert. 

        if (grant.cliff > block.number) revert AVV_CliffNotReached(grant.cliff);

But the `check` is a redundant one, since the `_getWithdrawableAmount` function call performs the same check and `returns 0` if the `grant.cliff > block.number` is satisfied. Then the `withdrawable` amount wil be `0`.

The follwoing condition will revert as a result since the `amount > 0`.

        if (amount > withdrawable) revert AVV_InsufficientBalance(withdrawable);

Hence the `grant.cliff > block.number` conditional check at the beginning of the `ARCDVestingVault.claim` function can be omitted to save gas.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L234-L238

## 6. REDUNDANT ASSIGNMENT OPERATION CAN BE OMITTED TO SAVE GAS ON AN EXTRA `MSTORE` OPERATION

The `ARCDVestingVault.claim` function is used by the `grant owners` to `claim` withdrawable value from a grant. In the `calim` function the following conditional checks are used to update the `grant.withdrawn` value and transfer the `withdrawable` amount to the `msg.sender`.

        if (amount == withdrawable) {
            grant.withdrawn += uint128(withdrawable);
        } else {
            grant.withdrawn += uint128(amount);
            withdrawable = amount;
        }

        ...

        token.safeTransfer(msg.sender, withdrawable);

The way in which the `withdrawable` variable is used above, can be changed to omit the `withdrawable = amount` `MSTORE` operation and to save gas as shown below:

        if (amount == withdrawable) {
            grant.withdrawn += uint128(amount);
        } else {
            grant.withdrawn += uint128(amount);
        }

        ...

        token.safeTransfer(msg.sender, amount);

The above modified code snippet does not implement the `withdrawable = amount` assignment and still performs the same logical operation as the previous code snippet, thus saving the gas on one `MSTORE`.  

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L241-L246
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L252