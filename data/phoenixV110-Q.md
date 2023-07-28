# [01] Unwanted blocking of gscAllawance for 7 days
 Currently, If newAllowance == gscAllawance[token] then the processing will happen and the lastAllowanceSet(token) is be updated to current timestamp. Which will lock the allowance for another 7 days.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L303-L323

## Recommendation
Add check if newAllowance == gscAllawance[token] then revert the transaction.

# [02] Check balance before token transfer
There is no check if the ether or ERC20 token balance of contract is greater than the amount or not. Need to be added to gracefully handle those cases.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L358-L373
 
## Recommendation
Add amount <= balance(address(this)) and amount <= token.balanceOf(address(this) checks. Otherwise revert the transaction.

# [03] Wrong parameter sent to an event
Wrong parameters has been sent to from and to parameters of event VoteChange(address indexed from, address indexed to, int256 amount). The parameters should be interchanged.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L148  

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L269

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L281C1-L281C1

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L356

## Recommendation
The event values should be as follows:

emit VoteChange(who, grant.delegatee, int256(uint256(newVotingPower)));
emit VoteChange(msg.sender, grant.delegatee, -1 * int256(grant.latestVotingPower));
emit VoteChange(msg.sender, to, int256(uint256(grant.latestVotingPower)));
emit VoteChange(who, grant.delegatee, change);

# [04] Unchecked amount deposit using transferFrom
The deposit function updated the amount variable but doesn’t checks the return bool of transferFrom() function.

 https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L197-L202

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L657  ## Recommendation

## Recommendation
Instead safeTransferFrom() should be used.
 
# [05] Redundant logic can be removed There is a redundant logic in the claim() method which can be removed as it is not providing any functionality as such.

  https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L241-L252

## Recommendation
The if else block can be removed and amount can be used for the logic.

grant.withdrawn += uint128(amount);

 // update the user's voting power
 _syncVotingPower(msg.sender, grant);

// transfer the available amount
token.safeTransfer(msg.sender, amount);  

# [06] Negatives should be handled before setting to uint variable
There is an instance in _syncVotingPower() method where it is assumed that the change will be negative and it should be multiplied by -ve sign. 

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L352

## Recommendation
An if check can be added which will multiply check by -1 if it is -ve to about underflows.  

# [07] Divide by 0 unhandled
In case the grant has been deleted grant.expiration and grant.cliff will be 0 which will make totalBlocksPostCliff 0. Which is further used as the denominator in the calculation.  

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L330 

## Recommendation
Add a check to ensure (grant.expiration - grant.cliff) > 0 

# [08] Inaccurate data in queryVotePowerView() method
There should be a check in queryVotePowerView() method that the input blockNumber should be greater than block.number - staleBlockLag. Because the queryVotePower() removes the data of blockNumbers less than block.number - staleBlackLag. 

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L112-L118

 ## Recommendation There should be a check to revert the transaction rather than sending 0. 

# [09] The checks should be added before data retrieval from storage
Before fetching the multiplierData for the tokenId and tokenAddress. The zero address and zero tokenId should be checked.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L418-L427

## Recommendation
Add the (tokenAddress == address(0) || tokenId == 0) at the beginning of the function.

# [10] Check value before execution
Check if the newBaseURI == baseURI otherwise set and send event.

  https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L57-L61  

## Recommendation 
Add a check if newBaseURI == baseURI.

# [11] Amount greater than 0 check missing in mint() method
In the below mint method the amount > 0 check is missing. The while method logic is being processed.

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L98-L120

## Recommendation
Add a check in the beginning of the method to ensure amount > 0.

