## 1. USE `safeTransferFrom` inplace of `transferFrom` OR ELSE CHECK THE SUCCESS OF THE RETURN VALUE

In the `NFTBoostVault._lockTokens` function the `transferFrom` is used to transfer the voting token from the `user` to the `NFTBoostVault` contract itself. But it is recommended to use `safeTransferFrom` inplace of `transferFrom` or else to check the `return` boolean value for success to verify that the transactino will suceesful. 

This is because in some `ERC20` tokens the `transferFrom` will `not revert` and will return `false` if the transfer fails.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L657
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L201

## 2. EMIT BOTH OLD AND NEW VALUES IN EVENTS WHEN CHANGING CRITICAL STATE VARIABLES

The `NFTBoostVault.setAirdropContract` is `onlyManager` modifier controlled function which the manager can use to change the `airdrop contract`. There is an event emitted once the contract is changed as follows:

        emit AirdropContractUpdated(newAirdropContract);

But it only refers to the new address and old address is ignored. It is recommneded to inlcude both the old and new address when emitting events for critical state variable changes.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L395
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L60
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L136
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L78

## 3. RECOMMENDED TO USE `call() function` WITH `CEI` AND `reentrancy guard` WHEN SENDING `eth` VIA PAYABLE ADDRESS

In the `ArcadeTreasury._spend` function, the `transfer` function is used to transfer `eth` as follows:

        if (address(token) == ETH_CONSTANT) {
            // will out-of-gas revert if recipient is a contract with logic inside receive()
            payable(destination).transfer(amount);
        }

But when using the `transfer` function, thier transaction gas is limited to `2300`. Hence this could lead to `DoS` if the recepient is a contract and performs additional compuation inside the `receive` function. 
Hence it is recommended to use the low level `call()` function with `CEI` and `reentrancy gaurd`.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L367
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L171

## 4. ERRORNEOUS `NATSPEC` COMMENTS SHOULD BE CORRECTED

        // check that after processing this we will not have `spent` more than the block limit
        Above should be corrected as follows:
        // check that after processing this we will not have `approved` more than the block limit        

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L385    

The following `Natspec` comments in the `ArcadeTokenDistributor` contract should be interchanged:

    /// @notice A flag to indicate if the launch partners have already been transferred to
    bool public vestingTeamSent;

    /// @notice A flag to indicate if the Arcade team has already been transferred to
    bool public vestingPartnerSent;

They should be corrected as follows:

    /// @notice A flag to indicate if the Arcade team has already been transferred to
    bool public vestingTeamSent;

    /// @notice A flag to indicate if the launch partners have already been transferred to
    bool public vestingPartnerSent;

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L53-L54
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L58-L59

## 5. LACK OF CONTRACT EXISTENCE CHECK BEFORE CALLING THE LOW LEVEL `call` FUNCTION

The `ArcadeTreasury.batchCalls` function is used to exeucte the low level `call` function to execute arbitary calls from the treasury in a batch. The low level `call` function with the `calldata` payload is exeucted on the destination address as shown below:

            (bool success, ) = targets[i].call(calldatas[i]);

But there is no check to make sure that the `targets[i]` has contract code in the destination address.
If the low level `call` function is executed on a destination contract without contract code the transaction will return `true` as the `success` bool value, even though the transaction did not execute. 

Hence it is recommended to check the contract code size as shown below before calling the low level `call` function. The following code snippet should be run before calling the low level `call` function inside the `ArcadeTreasury.batchCalls` function.

        uint256 codeSize;
        assembly {
            codeSize := extcodesize(targets[i])
        }
        require(codeSize > 0, "Can not transfer to contract with no contract code");

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341

## 6. USE UPPERCASE LETTERS TO DENOTE `constants` 

In the `ArcadeTokenDistributor` there are multiple constants declared. These constants are used to define distribution token amounts to respective parties. But these constants are denoted in lowercase letters. It is recommended to use uppercase letters to denote the `constants` as it is the accepted standard. This will further increase the readabilitiy of the code.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L32-L57