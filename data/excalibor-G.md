
# Gas Report

1. Compute hashing off-chain to save gas at deployment
[Here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L44C1-L46C83) we can compute these values off-chain to save gas at deployment as hashing is expensive for example:
```solidity
bytes32  public  constant ADMIN_ROLE =  0xdf8b4c520ffe197c5343c6f5aec59570151ef9a492f2c624fd45ddde6135ec42;
bytes32  public  constant BADGE_MANAGER_ROLE =  0x9605363ac419df002bafe0cc1dcbfb19cf2a2e2afa1c7428b115d0be2ecd78e5;
bytes32  public  constant RESOURCE_MANAGER_ROLE =  0xa08df8e9779f89161e9d4aa6eaffa8e1f95bfbc9b9812ee5dec3c3b19090625d;
```
2. Optimize for loops by utilizing unchecked math and no `i` instantiation
[here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L144) we can optimize this for loop by rewriting it like this
```solidity
for(uint i; i< _claimData.length; ) {
	// Logic
	unchecked { ++i; }
}
```
3. State variable unnecessary down-sizing
All uint variables are by default set to `uint256` so turning [this](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L55) into a uint48 actually costs extra gas as the EVM has to down-size for every interaction. Another instance [here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L59)
4. Don't down-size `ClaimData` variables
[Here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/interfaces/IReputationBadge.sol#L10C4-L15C6) we should avoid down-sizing these values to save gas, due to the fact this data is only read in `publishRoots` its less efficient to pack these variables since we are calling from a calldata array, the extra computation to down-size outweighs leaving `claimExpiration` at `uint256`
```solidity
struct  ClaimData {
	uint208 tokenId;
	uint48 claimExpiration;
	bytes32 claimRoot;
	uint256 mintPrice;
}
```
| Method | Avg Cost before changes | Avg Cost after changes | Gas savings
|:----------| :----------| :----------| :----------| 
| publishRoots | 138021 | 137077 | 944 |

5. Pack state variables to save gas
[These](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L93C1-L97C40) variables can be packed into one storage slot to save gas
```solidity
    address public minter;
    uint96 public mintingAllowedAfter;
```

as `mintingAllowedAfter` is a timestamp variable we can safely pack it into a uint96 as this will not overflow for thousands of years.

6. Remove reentrancy guard and refactor code so its not vulnerable to reentrancy
[Here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L108C1-L120C6) we have an unnecessary reentrancy guard we can make a refactor as follows:

```solidity
function  gscSpend(
address  token,
uint256  amount,
address  destination
) external  onlyRole(GSC_CORE_VOTING_ROLE) {
	if (destination ==  address(0)) revert  T_ZeroAddress("destination");
	if (amount ==  0) revert  T_ZeroAmount();

	// Will underflow if amount is greater than remaining allowance
	gscAllowance[token] -= amount;

	_spend(token, amount, destination, spendThresholds[token].small);
}
```
The same can be done for the below functions that call `_spend()` as well.

7. Pack the `Grant` struct into fewer slots to save gas
 [Here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/libraries/ARCDVestingVaultStorage.sol#L29C2-L38C6) we can pack some of these smaller variables to save gas here is a recommended example of how to pack them:
 ```solidity
 struct  Grant {
	uint128 allocation;
	uint128 cliffAmount;
	uint128 withdrawn;
	uint128 created;
	uint256 latestVotingPower;
	address delegatee;
	uint48 expiration;
	uint48 cliff;
}
```

This change packs the struct into 4 slots where it was 5 before, we can do this as `expiration` and `cliff` are timestamp variables, which means they will not feasibly overflow at this sizing.

8. Wasted SSTORE calls when revoking a grant
[Here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L164C1-L172C51) we are constantly updating `grant.withdrawn` just to set it to zero at the end of the call, this can be refactored to not waste SSTORE calls

```solidity
function  revokeGrant(address  who) external  virtual  onlyManager {
	// Logic
	
unchecked {
	uint256 cachedWithdrawn = grant.withdrawn;
	uint256 withdrawable =  _getWithdrawableAmount(grant);
	token.safeTransfer(who, withdrawable);

	// transfer the remaining tokens to the vesting manager
	uint256 remaining = grant.allocation - (cachedWithdrawn + withdrawable);
	token.safeTransfer(msg.sender, remaining);
	grant.withdrawn =  uint128(cachedWithdrawn + remaining + withdrawable);
}

	// Remaining logic
}
```
Due to the nature of how `grant.allocation` will always be greater then `(grant.withdrawn + withdrawable)` we can safely wrap it all in an unchecked block

9.  Use unchecked math when we ensure it is safe
There are many instances of this but [here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L213C1-L215C35) is one example in this instance we check `x > y` and then we do `x-y` since we confirm that x is always greater then y `x-y` will never underflow so we can safely wrap it in an unchecked block.

```solidity
if (unassigned.data < amount) revert  AVV_InsufficientBalance(unassigned.data);

unchecked {
	unassigned.data -= amount;
}
```
10. Keep constant variables in the default 256 bit size
[Here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L66C1-L69C58) we convert these constant variables to uint128 unnecessarily, we can keep them as uint256 to save some gas. uint256 is the default size so it costs extra to down-size, in this instance it does not benefit us to do so.

11. Make internal functions use memory to compute instead of a storage pointer to avoid unnecessary storage calls
[These three functions](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L579C4-L637C6) can be refactored to pass down variables and use memory  instead of doing all the math with a storage pointer. This will significantly reduce the amount of SLOAD calls we make

Example Refactors: 
```solidity
function  _syncVotingPower(address  who, uint128  latestVotingPower, uint128  amount, uint128  withdrawn, address  tokenAddress, uint128  tokenId, address  delegatee) internal  returns(uint128) {
	
	History.HistoricalBalances memory votingPower =  _votingPower();
	uint256 delegateeVotes = votingPower.loadTop(delegatee);

	uint256 newVotingPower =  _currentVotingPower(amount, withdrawn, tokenAddress, tokenId);

	// get the change in voting power. Negative if the voting power is reduced
	int256 change =  int256(newVotingPower) -  int256(uint256(latestVotingPower));

	// do nothing if there is no change
	if (change ==  0) return latestVotingPower;
	if (change >  0) {
		votingPower.push(delegatee, delegateeVotes +  uint256(change));
	} else {
		// if the change is negative, we multiply by -1 to avoid underflow when casting
		votingPower.push(delegatee, delegateeVotes -  uint256(change *  -1));
	}

	latestVotingPower =  uint128(newVotingPower);
	emit  VoteChange(who, delegatee, change);
	return latestVotingPower;
}

  
  

function  _getWithdrawableAmount(
uint128  withdraw,
uint128  amount
) internal  pure  returns (uint256) {
	if (withdrawn == amount) {
		return  0;
	}

	return amount - withdrawn;

}

  
  

function  _currentVotingPower(
uint128  amount,
uint128  withdrawn,
address  tokenAddress,
uint128  tokenId
) internal  view  virtual  returns (uint256) {
	uint128 locked = amount - withdrawn;

	if (tokenAddress !=  address(0) && tokenId !=  0) {
		return (locked *  getMultiplier(tokenAddress, tokenId)) / MULTIPLIER_DENOMINATOR;
	}

	return locked;
}
```

This applies to a lot of internal functions as well.

Another refactor we can make to `_syncVotingPower` is to not pass down the `who` variable as its only used to emit the event, instead we can emit the event in the external function which would call `_syncVotingPower`.

12. Unnecessary `_withdrawNft()` call is wasting gas
[Here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L316C1-L320C10) the `_withdrawNft()` call also wastes a lot of gas because of the interactions with storage we can safely remove [this](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L316C1-L320C10) whole code snippet to save gas, instead we can replace it with this
```solidity
if (registration.tokenAddress !=  address(0) && registration.tokenId !=  0) {

	// withdraw the current ERC1155 from the registration
	IERC1155(registration.tokenAddress).safeTransferFrom(
	address(this),
	msg.sender,
	registration.tokenId,
	1,
	bytes("")
	);

}
```
This is because `_withdrawNft` makes a lot of unnecessary state changes for this function.

| Method | Avg Cost before changes | Avg Cost after changes | Gas savings
|:----------| :----------| :----------| :----------| 
| updateNft | 155654 | 147743 | 7911 |

13. Redundant check inside `_withdrawNft` can be removed to save gas
[Here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L247C1-L249C14) we check for the same conditions that `_withdrawNft` already checks for. So we can remove [the check](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L552C1-L553C84) inside `_withdrawNft` to not have a redundant check. We also need to update to the check inside the external function `withdrawNft` [here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L294) so it is still safe.

14. `balance` is an unnecessary  storage variable
The ERC20 standard already saves user balances and is tracking the balance of the contract so whenever a user is depositing/withdrawing tokens we are calling extra SSTORE/SLOADS for no reason I am referencing [this](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/BaseVotingVault.sol#L143C1-L150C6).

	As a note this is not completely unnecessary because if a user sends tokens to the contract manually we will not be able to track which are deposited through function calls of the vault and which were sent manually, alternatively we can add a function like `claimDust()` or `getDust()` to see if there are tokens that were sent be accident, as we do not have this functionality currently, it is safe to remove the storage variable. 

15. Cache storage data when reading from it multiple times to save gas by reducing SLOAD calls
For example [here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L182C5-L212C6) by caching `registration.delegatee` and `registration.latestVotingPower` we can save a lot of gas because of the repetitive SLOAD calls, here is an example refactor: 
```solidity
function  delegate(address  to) external  override {
	
	if (to ==  address(0)) revert  NBV_ZeroAddress("to");

	NFTBoostVaultStorage.Registration storage registration =  _getRegistrations()[msg.sender];

	uint256 _latestVotingPower = registration.latestVotingPower;

	address _oldDelegate = registration.delegatee;

	// If to address is already the delegate, don't send the tx
	if (to == _oldDelegate) revert  NBV_AlreadyDelegated();	  


	History.HistoricalBalances memory votingPower =  _votingPower();

	uint256 oldDelegateeVotes = votingPower.loadTop(_oldDelegate);	  


	// Remove voting power from old delegatee and emit event
	votingPower.push(_oldDelegate, oldDelegateeVotes - _latestVotingPower);

	emit  VoteChange(msg.sender, _oldDelegate, -1  *  int256(uint256(_latestVotingPower)));  


	// Note - It is important that this is loaded here and not before the previous state change because if
	// to == registration.delegatee and re-delegation was allowed we could be working with out of date state
	uint256 newDelegateeVotes = votingPower.loadTop(to);


	// return the current voting power of the Registration. Varies based on the multiplier associated with the
	// user's ERC1155 token at the time of txn
	uint256 addedVotingPower =  _currentVotingPower(registration.amount, registration.withdrawn, registration.tokenAddress, registration.tokenId);	  

	// add voting power to the target delegatee and emit event
	votingPower.push(to, newDelegateeVotes + addedVotingPower);	  

	// update registration properties
	registration.latestVotingPower =  uint128(addedVotingPower);
	registration.delegatee = to;	  
	
	emit  VoteChange(msg.sender, to, int256(addedVotingPower));
}
```
| Method | Avg Cost before changes | Avg Cost after changes | Gas savings
|:----------| :----------| :----------| :----------| 
| delegate | 118171 | 117692 | 479 |
