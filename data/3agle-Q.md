# QA Report for Arcade.xyz

## Overview
During the audit, 5 Low, 2 Refactoring, and 2 Non-Critical issues were found.

### Low-Risk Issues

|#|Issue|Instances|
|-|:-|:-:|
| [L-01] | Missing validation that grant exists in `ARCDVestingVault` | 1 |
|[L-02]|Check if the `msg.sender` has the token `amount`|1|
|[L-03]|Ensure that registration exists in `NFTBoostVault`|1|
|[L-04]|`nonReentrant` modifier is not present for `NFTBoostVault::delegate`|1|
|[L-05]|Single-Step Ownership Transfers|2|

**Total** 6 instances over 5 issues

### Refactoring Issues

|#|Issue|Instances|
|-|:-|:-:|
| [R-01]| Redundant Check in `withdraw` and `updateNft` Functions of `NFTBoostVault`  | 2 |
| [R-02]| Redundant Logic in ARCDVestingVault's `claim` Function | 1 |

**Total** 3 instances over 2 issues

### Non-critical Issues

|#|Issue|Instances|
|-|:-|:-:|
| [NC-01]| Check for `ETH_CONSTANT` in `ArcadeTreasury::approve` | 1 |
| [NC-02]| Use of Redundant Parameter in `NFTBoostVault::_lockNft`  | 1 |

**Total** 2 instances over 2 issues

## Low-Risk Issues (5)

### [L-01] Missing check that grant exists in `ARCDVestingVault` 

**Description**
- The audited contract does not appear to have a necessary check for whether a grant exists within the `ARCDVestingVault::delegate()` function.

**Instances**
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L260-L282
- This function attempts to update the caller's voting power delegatee but fails to first check whether a grant is in existence.
- Here's the block of code presenting the issue:
```solidity
function delegate(address to) external {
    ARCDVestingVaultStorage.Grant storage grant = _grants()[msg.sender];
    if (to == grant.delegatee) revert AVV_AlreadyDelegated();
    ....
    grant.delegatee = to;
    ....
}
```

**Recommendation**
- It's suggested to add an extra layer of validation to check if the `allocation` attribute in the `grant` object is equal to zero. If it's zero (which means that no grant is defined), the function should stop executing the function there and revert the transaction.
- Here's an example:
```solidity
function delegate(address to) external {
    ARCDVestingVaultStorage.Grant storage grant = _grants()[msg.sender];
    if (grant.allocation == 0) revert AVV_NoGrantSet();
    if (to == grant.delegatee) revert AVV_AlreadyDelegated();
    ....
    grant.delegatee = to;
    ....
}
```

### [L-02] Check if the `msg.sender` has the token `amount`

**Description**
- The `NFTBoostVault::addTokens()` does not check whether the `msg.sender` has enough tokens in their balance before proceeding to add tokens.

**Instances**
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L266-L287
- This function aims to add a specific amount of tokens to the contract's balance without first verifying if the `msg.sender` has enough tokens to transfer.
- Here's the block of code presenting the issue:
```solidity
function addTokens(uint128 amount) external override nonReentrant {
    if (amount == 0) revert NBV_ZeroAmount();
    ...
    // transfer user ERC20 amount into this contract
    _lockTokens(msg.sender, amount, address(0), 0, 0);
}
```

**Recommendation**
- You need to add an additional check before transferring the token to the contract.
```solidity
function addTokens(uint128 amount) external override nonReentrant {
    if (amount == 0) revert NBV_ZeroAmount();

    // Check if the message sender has the token amount
    if (token.balanceOf(msg.sender) < amount ) revert NBV_NotEnoughTokens();

    // load the registration
    NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

    ...

    // transfer user ERC20 amount into this contract
    _lockTokens(msg.sender, amount, address(0), 0, 0);
}
```

### [L-03] Absence of Existence Check for `NFTBoostVault` Registration

**Description**
- A crucial oversight is observed in the `delegate` function of the audited smart contract. The function currently neglects to ensure registration exists within the `NFTBoostVault`, allowing non-registered users to delegate to an address.
- This also allows bypassing all the checks in the following function which checks the existence of a registration through the `registration.delegatee` parameter leading to unforeseen issues:
	- `addTokens()`
	- `updateNft()`

**Instances**
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L182-L212
- This is evident in the provided instance of the `delegate` function, which changes the voting power delegatee but does not validate registration's existence beforehand.
- The code snippet below reveals the problem:
```solidity
function delegate(address to) external override {
    if (to == address(0)) revert NBV_ZeroAddress("to");

    NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

    // If to address is already the delegate, don't send the tx
    if (to == registration.delegatee) revert NBV_AlreadyDelegated();

    // ... Remaining method logic
}
```
**Recommendation**
- Recommended best practice entails conducting an existence check for the inbound registration prior to any additional operations.
- Amend the function to include the following additional check:
```solidity
function delegate(address to) external override {
    if (to == address(0)) revert NBV_ZeroAddress("to");

    NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

    // Check if registration exists  
    if (registration.delegatee == address(0)) revert NBV_NoRegistration();

    // If to address is already the delegate, don't send the tx
    if (to == registration.delegatee) revert NBV_AlreadyDelegated();

    // ... Remaining method logic
}
```

### [L-04] `nonReentrant` modifier is not present for `NFTBoostVault::delegate`

**Description**
- The  `NFTBoostVault::delegate()` function lacks the essential `nonReentrant` modifier.
**Instances**
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L182
- Here is the current version of the function:
```solidity
function delegate(address to) external override {
    if (to == address(0)) revert NBV_ZeroAddress("to");

    NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

    // If to address is already the delegate, don't send the tx
    if (to == registration.delegatee) revert NBV_AlreadyDelegated();

    // ... Rest of the code
}
```
**Recommendation**
- It is essential to add the `nonReentrant` modifier to the `delegate` function to ensure it cannot be re-entered while still executing.
- By adding the `nonReentrant` modifier, you block possible cross-function and cross-contract re-entrancies, enhancing the contract security.
- The function should be updated as follows:
```solidity
function delegate(address to) external override nonReentrant {
    if (to == address(0)) revert NBV_ZeroAddress("to");

    NFTBoostVaultStorage.Registration storage registration = _getRegistrations()[msg.sender];

    // If to address is already the delegate, don't send the tx
    if (to == registration.delegatee) revert NBV_AlreadyDelegated();

   // ... Rest of the code
}
```
### [L-05] Single-Step Ownership Transfers

**Description**
- Two identified instances within `ArcadeToken.sol` and `BaseVotingVault.sol` exhibit single-step ownership transfers.

**Instances** 
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L132-L136
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L68-L72
- This issue is evident in the `setMinter` function in `ArcadeToken.sol` and the `setTimelock` function in `BaseVotingVault.sol`.
- Here's how they currently look:

**ArcadeToken.sol**:
```solidity
function setMinter(address _newMinter) external onlyMinter {
    if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

    minter = _newMinter;
    emit MinterUpdated(minter);
}
```

**BaseVotingVault.sol**:
```solidity
function setTimelock(address timelock_) external onlyTimelock {
    if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");

    Storage.set(Storage.addressPtr("timelock"), timelock_);
}
```

- In both of these functions, the assignment of a new owner/minter is immediate and irrevocable, done via a single method call. This one-step transfer might be insecure as it does not provide the new owner the chance to confirm that they can indeed act as the new owner/minter.

**Recommendation**
- Implementing a two-step ownership transfer is recommended for increasing security. In this concept, after the current owner initiates the transfer, the new owner has to affirmatively accept the ownership.
- Here's an example of how a two-step ownership transfer could be implemented:
```solidity
// Owner proposes the new owner
function proposeNewOwner(address _newOwner) external onlyOwner {
    proposedOwner = _newOwner;
}
// Proposed owner accepts the new ownership
function acceptOwnership() external {
    require(msg.sender == proposedOwner, "Only proposed owner can accept ownership");
    emit OwnershipTransferred(owner, proposedOwner);
    owner = proposedOwner;
}
```

- In the above code snippet, a new function `proposeNewOwner` is introduced which can only be called by the current owner. It sets the `proposedOwner` of the smart contract. Simultaneously, a corresponding `acceptOwnership` function needs to be included which allows the proposed owner to accept the ownership transfer.
- This two-step model provides an additional security layer and is safer. Applying this suggestion helps ensure contract robustness and safety against unexpected ownership assignments.

## Refactoring Issues (2)

### [R-01] Redundant Check in `withdraw` and `updateNft` Functions of `NFTBoostVault`

**Description**
- The smart contract `NFTBoostVault` performs redundant checks in its `withdraw` and `updateNft` functions for non-zero token addresses and token IDs. 
- These duplicate checks are unserviceable as they are already implemented in the `_withdrawNft` internal function that both functions call.

**Instances**
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L247
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L317
- The issue is found in the `withdraw` and `updateNft` functions in the `NFTBoostVault` contract:
```solidity
// In withdraw function
if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
  _withdrawNft();
}

// In updateNft function
if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
  _withdrawNft();
}
```
- These checks are already present in the _withdrawNft() function
```solidity
    function _withdrawNft() internal {
        // existing code omitted for brevity

        if (registration.tokenAddress == address(0) || registration.tokenId == 0)
            revert NBV_InvalidNft(registration.tokenAddress, registration.tokenId);

        // existing code omitted for brevity
    }
```

**Recommendation**
- It's best to remove these redundant checks from the `withdraw` and `updateNft` functions, thereby leaning on the validation already taking place in the `_withdrawNft` function. 
- The revised functions should be as follows:

**For `withdraw` function:**
```solidity
if (registration.withdrawn == registration.amount) {
  _withdrawNft();

  // delete registration
  registration.amount = 0;
  registration.latestVotingPower = 0;
  registration.withdrawn = 0;
  registration.delegatee = address(0);
}
```

**For `updateNft` function:**
```solidity
// if the user already has an ERC1155 registered, withdraw it
_withdrawNft();

// set the new ERC1155 values in the registration and lock the new ERC1155
registration.tokenAddress = newTokenAddress;
registration.tokenId = newTokenId;

```

- By incorporating these suggested amendments, the code is made cleaner, and future maintainers of this code do not get perplexed by unnecessary validations. 
- It also marginally improves the contract's efficiency by minimizing redundant operations during code execution.

### [R-02] Redundant Logic in ARCDVestingVault's `claim` Function

**Description**
- In the smart contract `ARCDVestingVault`, the `claim` function has a complex and redundant logic while handling the claim operation. The function currently checks if the `amount` is equal to `withdrawable` and then accordingly adjusts the `grant.withdrawn` and `withdrawable` values. 
- This redundant comparison increases unnecessary complexity.

**Instances**
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L241-L246
- The issue is found in the `claim` function in the `ARCDVestingVault` contract:
```solidity
function claim(uint256 amount) external override nonReentrant {
    // existing code omitted for brevity

    // update the grant's withdrawn amount
    if (amount == withdrawable) {
        grant.withdrawn += uint128(withdrawable);
    } else {
        grant.withdrawn += uint128(amount);
        withdrawable = amount;
    }

    // existing code omitted for brevity
}
```

**Recommendation**
- It is recommended to simplify this logic by directly storing the `amount` in `grant.withdrawn` and then passing it to the `token.safeTransfer` function. Below is an example of how this adjustment could look:
```solidity
function claim(uint256 amount) external override nonReentrant {
    // existing code omitted for brevity

    // update the grant's withdrawn amount
    grant.withdrawn += uint128(amount);

    // existing code omitted for brevity 

    // transfer the available amount
    token.safeTransfer(msg.sender, amount);
}
```
- By doing this, you achieve cleaner and more efficient code that eliminates an unnecessary variable and simplifies the reading and understanding of the contract for others.
## Non-critical Issues (2)

### [NC-01] Check for `ETH_CONSTANT` in ArcadeTreasury's `approve`

**Description**
- In the audited smart contract `ArcadeTreasury`, the `_approve` function lacks a crucial check for `ETH_CONSTANT`. The contract uses addresses of tokens for permitting spending and approvals. 
- It also uses an address to signify ETH (`ETH_CONSTANT = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE)`).
- However, while a corresponding check is present in a function for spending (`_spend`), it is not included in the approval function (`_approve`), despite there being no ERC20 contract for ETH. As it stands, the `_approve` function reverts when passed `ETH_CONSTANT`.

**Instances**
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L384-L394
- The issue is found in the `_approve` function in the `ArcadeTreasury` contract:
```solidity
function _approve(address token, address spender, uint256 amount, uint256 limit) internal {
    // check that after processing this we will not have spent more than the block limit
    uint256 spentThisBlock = blockExpenditure[block.number];
    if (amount + spentThisBlock > limit) revert T_BlockSpendLimit();
    blockExpenditure[block.number] = amount + spentThisBlock;

    // approve tokens
    IERC20(token).approve(spender, amount);

    emit TreasuryApproval(token, spender, amount);
}
```

**Recommendation**
- It is suggested to add a respective check that will revert when `ETH_CONSTANT` is passed to the `_approve` function, similar to the check in the `_spend` function.
- The updated code would look like this:
```solidity
function _approve(address token, address spender, uint256 amount, uint256 limit) internal {
    // check that after processing this we will not have spent more than the block limit
    uint256 spentThisBlock = blockExpenditure[block.number];
    if (amount + spentThisBlock > limit) revert T_BlockSpendLimit();
    blockExpenditure[block.number] = amount + spentThisBlock;

    // revert if token is ETH_CONSTANT as approve operation is not valid for ETH
    if (token == ETH_CONSTANT) revert T_InvalidOperationForETH();

    // approve tokens
    IERC20(token).approve(spender, amount);

    emit TreasuryApproval(token, spender, amount);
}
```
- By adding these checks where necessary, it helps to prevent unintended and unexpected results from function calls, ensuring the contract behaves as intended.


### [NC-02] Use of Redundant Parameter in `NFTBoostVault::_lockNft`

**Description**
- In the smart contract `NFTBoostVault`, the `_lockNft` function includes a parameter `nftAmount`, which is redundant. The protocol only locks a single ERC1155 token from a user at a time. 
- Consequently, `nftAmount` always equals `1`.

**Instances**
- https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L673-L675
- The issue is found in the `_lockNft` function in the `NFTBoostVault` contract:
```solidity
function _lockNft(address from, address tokenAddress, uint128 tokenId, uint128 nftAmount) internal {
    IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));
}
```

**Recommendation**
- It is recommended to pass a constant `1` instead of the `nftAmount` in the `safeTransferFrom` function, as the protocol locks exactly one ERC1155 token of a user at the time.
- Here's how can you update the `_lockNft` function code:
```solidity
function _lockNft(address from, address tokenAddress, uint128 tokenId) internal {
    IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, 1, bytes(""));
}
```
- Implementing this recommendation simplifies the function interfaces in this contract, making the function easier to understand and use. Also, it could potentially save a small amount of gas, as the operation and storage of redundant variables are eliminated. 
- Further, it enforces the policy more clearly that only one ERC1155 token is to be locked at the time, enhancing the overall security and correctness of the contract.