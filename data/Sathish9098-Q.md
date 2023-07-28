
# LOW FINDINGS

##

##

## [L-1] Use safeInceaseAllowance()/safeDecreaseAllowance() instead of approve() or deprecated safeApprove() Functions

### Impact
The ``safeIncreaseAllowance()`` and ``safeDecreaseAllowance()`` functions also prevent ``front-running attacks`` by atomically updating the allowance and emitting an Approval event. This means that the allowance cannot be changed by another transaction between the time the allowance is updated and the Approval event is emitted. 

The safeApprove() function was deprecated because it was not gas-efficient and it did not prevent front-running attacks. You should use safeIncreaseAllowance() or safeDecreaseAllowance() instead of safeApprove()

### POC

```solidity
FILE: Breadcrumbs2023-07-arcade/contracts/ArcadeTreasury.sol

200:  _approve(token, spender, amount, spendThresholds[token].small);

219: _approve(token, spender, amount, spendThresholds[token].small);

238:  _approve(token, spender, amount, spendThresholds[token].medium);

257: _approve(token, spender, amount, spendThresholds[token].large);

391:   IERC20(token).approve(spender, amount);

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L391

### Recommended Mitigation
Use ``safeIncreaseAllowance()`` or ``safeDecreaseAllowance() `` instead of approve() or safeApprove()

##

## [L-2] Don’t use payable.transfer()

### Impact
The use of ``payable.transfer()`` is [heavily frowned upon](https://consensys.io/diligence/blog/2019/09/stop-using-soliditys-transfer-now/) because it can lead to the locking of funds. The transfer() call requires that the recipient is either an EOA account, or is a contract that has a payable callback. For the contract case, the transfer() call only provides 2300 gas for the contract to complete its operations. This means the following cases can cause the transfer to fail:

- The contract does not have a payable callback
- The contract’s payable callback spends more than 2300 gas (which is only enough to emit something)

### POC

```solidity
FILE: 2023-07-arcade/contracts/ArcadeTreasury.sol

367: payable(destination).transfer(amount);

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L367

```solidity
FILE: 2023-07-arcade/contracts/nft/ReputationBadge.sol

171: payable(recipient).transfer(balance);

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L171

##

## [L-3] Hardcoded cooldown period may cause problem in future 

### Impact
Hardcoding a cooldown period in the contract may lead to potential problems in the future. The hardcoded value of ``SET_ALLOWANCE_COOL_DOWN`` is currently set to ``7 days``, which means the cooldown period for updating the GSC allowance is fixed and cannot be changed without redeploying the contract.

### POC

```solidity
FILE: 2023-07-arcade/contracts/ArcadeTreasury.sol

56: uint48 public constant SET_ALLOWANCE_COOL_DOWN = 7 days;

```

### Recommended Mitigation
Cooldown period can be set by the contract owner or by a governance mechanism. This allows the cooldown period to be adjusted as needed, without having to modify the contract code.

##

## [L-4] Lack of ``nonReentrant`` modifier in critical ``withdraw()``,``reclaim()`` functions

### Impact
The withdraw function should have a reentrancy modifier to protect against reentrancy attacks. Although the function already checks for the available balance (unassigned.data) and transfers the tokens using token.safeTransfer, it is still vulnerable to reentrancy attacks if the token.safeTransfer function itself or any other function it calls contains external contract calls

### POC

```solidity
FILE: 2023-07-arcade/contracts/ARCDVestingVault.sol

function withdraw(uint256 amount, address recipient) external override onlyManager {
        Storage.Uint256 storage unassigned = _unassigned();
        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
        // update unassigned value
        unassigned.data -= amount;

        token.safeTransfer(recipient, amount);
    }

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L211-L218

```solidity
FILE: Breadcrumbs2023-07-arcade/contracts/token/ArcadeAirdrop.sol

function reclaim(address destination) external onlyOwner {
        if (block.timestamp <= expiration) revert AA_ClaimingNotExpired();
        if (destination == address(0)) revert AA_ZeroAddress("destination");

        uint256 unclaimed = token.balanceOf(address(this));
        token.safeTransfer(destination, unclaimed);
    }

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeAirdrop.sol#L62

### Recommended Mitigation
Use ``nonReentrant`` to avoid reentrancy attacks. And this gives extra layer of safety .

##

## [L-5] The ``addGrantAndDelegate`` function not checked the new ``delegatee`` already has an active grant

### Impact
The function ``addGrantAndDelegate`` allows the manager to create a new grant without checking if the grant recipient already has an active grant. If a user already has an active grant, this function will overwrite the existing grant, which might lead to unintended behavior. It would be better to check if the recipient already has a grant and handle this situation accordingly.

### POC

```solidity
FILE: 2023-07-arcade/contracts/ARCDVestingVault.sol

function addGrantAndDelegate(
        address who,
        uint128 amount,
        uint128 cliffAmount,
        uint128 startTime,
        uint128 expiration,
        uint128 cliff,
        address delegatee
    ) external onlyManager {
        // input validation
        if (who == address(0)) revert AVV_ZeroAddress("who");
        if (amount == 0) revert AVV_InvalidAmount();

        // if no custom start time is needed we use this block.
        if (startTime == 0) {
            startTime = uint128(block.number);
        }
        // grant schedule check
        if (cliff >= expiration || cliff < startTime) revert AVV_InvalidSchedule();

        // cliff check
        if (cliffAmount >= amount) revert AVV_InvalidCliffAmount();

        Storage.Uint256 storage unassigned = _unassigned();
        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);

        // load the grant
        ARCDVestingVaultStorage.Grant storage grant = _grants()[who];

        // if this address already has a grant, a different address must be provided
        // topping up or editing active grants is not supported.
        if (grant.allocation != 0) revert AVV_HasGrant();

        // load the delegate. Defaults to the grant owner
        delegatee = delegatee == address(0) ? who : delegatee;

        // calculate the voting power. Assumes all voting power is initially locked.
        uint128 newVotingPower = amount;

        // set the new grant
        grant.allocation = amount;
        grant.cliffAmount = cliffAmount;
        grant.withdrawn = 0;
        grant.created = startTime;
        grant.expiration = expiration;
        grant.cliff = cliff;
        grant.latestVotingPower = newVotingPower;
        grant.delegatee = delegatee;

        // update the amount of unassigned tokens
        unassigned.data -= amount;

        // update the delegatee's voting power
        History.HistoricalBalances memory votingPower = _votingPower();
        uint256 delegateeVotes = votingPower.loadTop(grant.delegatee);
        votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);

        emit VoteChange(grant.delegatee, who, int256(uint256(newVotingPower)));
    }

```

### Recommended Mitigation
Add a check to ensure that the recipient does not already have an active grant

```solidity
 ARCDVestingVaultStorage.Grant storage existingGrant = _grants()[who];
 if (existingGrant.allocation != 0) revert AVV_HasGrant();

```

##

## [L-6] No same value input control in critical setMinter() functions

###
The same address value should be avoided 

### POC

```
FILE: 2023-07-arcade/contracts/token/ArcadeToken.sol

 function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

        minter = _newMinter;
        emit MinterUpdated(minter);
    }


```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132-L137

### Recommended Mitigation

Add same value input control 

```
require(_newMinter!=minter, "Same address value ")
```
##

## [L-7] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

##

## [L-8] ``Accidental`` token transfers can't be recovered from contract

### Impact
Accidental token transfers are typically irreversible and cannot be recovered by the contract. Once tokens are transferred out of a contract, they are under the control of the recipient, and the contract has no authority to reverse or undo the transaction.

### Recommended Mitigation

```solidity
function recoverToken(
        address tokenAddress,
        uint256 tokenId,
        address recipient,
        uint256 amount
    ) external onlyOwnerOrAuthorized {
        if (tokenAddress == address(0)) {
            // Recover ERC20 tokens
            IERC20(tokenAddress).transfer(recipient, amount);
        } else {
            // Recover ERC1155 NFTs
            IERC1155(tokenAddress).safeTransferFrom(address(this), recipient, tokenId, amount, bytes(""));
        }
    }
```

##

## [L-9] Lack of checks for critical ``baseURI`` string value 

### Impact

Empty base URI, which could lead to incorrect behavior when generating metadata URLs for ERC1155 NFTs.

The setBaseURI function lacks proper checks for critical operations on the baseURI string value. It's important to validate and handle inputs to prevent potential vulnerabilities

### POC

```solidity
FILE: 2023-07-arcade/contracts/nft/BadgeDescriptor.sol

  function setBaseURI(string memory newBaseURI) external onlyOwner {
        baseURI = newBaseURI;

        emit SetBaseURI(msg.sender, newBaseURI);
    }


```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/BadgeDescriptor.sol#L57-L61

### Recommended Mitigation

```solidity

 require(bytes(newBaseURI).length > 0, "Base URI cannot be empty");

```

##

## [L-10] No time lock mechanism to delay the distribution used critical toTreasury(),toDevPartner(),toCommunityRewards(),toCommunityAirdrop(), toTeamVesting(),toPartnerVesting() Functions 

### Impact
 If the functions is mistakenly called (e.g., due to human error or miscommunication) or maliciously triggered by an unauthorized entity, the tokens are transferred instantly to the addresses. Once the transfer is executed, it is irreversible, and there's no way to recover the tokens unless the receiver address voluntarily returns them.

### POC

https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeTokenDistributor.sol#L72-L164

### Recommended Mitigation
The contract owner might set a time locks

##

## [L-11] Don't use ERC1155 

### Impact
The main limitation of ERC1155 is the lack of individual scarcity for each token. Since ERC1155 tokens can represent multiple types of assets, they do not possess the same level of uniqueness as ERC721 tokens .

### POC

```solidity
FILE: Breadcrumbs2023-07-arcade/contracts/NFTBoostVault.sol

5: import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";

```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/NFTBoostVault.sol#L5

### Recommended Mitigation
Use ERC721 instead ERC1155 




