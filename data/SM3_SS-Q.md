
#  Low Risk Issues

| Number | Issues | Instances |
|:--------:|:--------:|:-----------:|
|[L-01]| Array lengths not checked | 4 |
|[L-02]| Functions calling contracts/addresses with transfer hooks are missing reentrancy guards | 13 |
|[L-03]| _grantVotingPower() does note remove old entries before adding new ones | 1 |
|[L-04]| Using zero as a parameter  | 3 |
|[L-05]| address shouldn't be hard-coded  | 1 |
|[L-06]| Consider implementing two-step procedure for updating protocol addresses | 4 |



## [L‑01] Array lengths not checked

If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array

```solidity
file:   contracts/ArcadeTreasury.sol

333         function batchCalls(
        address[] memory targets,
        bytes[] calldata calldatas
    ) external onlyRole(ADMIN_ROLE) nonReentrant 


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L333-L336

```solidity
file:    contracts/nft/ReputationBadge.sol

98        function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable

140       function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) 

204        function _verifyClaim(
        address recipient,
        uint256 tokenId,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) internal view returns (bool)


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L98-L105

## [L‑02] Functions calling contracts/addresses with transfer hooks are missing reentrancy guards

Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to read-only reentrancies with no way to protect against it, except by block-listing the whole protocol.

There are 13 instances of this issue:


```solidity
file:  contracts/ArcadeTreasury.sol

367     payable(destination).transfer(amount);

369    IERC20(token).safeTransfer(destination, amount);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L367

```solidity
file:  contracts/ARCDVestingVault.sol

167    token.safeTransfer(who, withdrawable);

172    token.safeTransfer(msg.sender, remaining);

201    token.transferFrom(msg.sender, address(this), amount);

217    token.safeTransfer(recipient, amount);

252    token.safeTransfer(msg.sender, withdrawable);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L167

```solidity
file:  contracts/NFTBoostVault.sol

258    token.safeTransfer(msg.sender, amount);

556             IERC1155(registration.tokenAddress).safeTransferFrom(
            address(this),
            msg.sender,
            registration.tokenId,
            1,
            bytes("")
        );

657    token.transferFrom(from, address(this), amount);

674    IERC1155(tokenAddress).safeTransferFrom(from, address(this), tokenId, nftAmount, bytes(""));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L258

```solidity
file:  contracts/token/ArcadeAirdrop.sol

67   token.safeTransfer(destination, unclaimed);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L67

```solidity
file: contracts/token/ArcadeTokenDistributor.sol

79   arcadeToken.safeTransfer(_treasury, treasuryAmount);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L79



## [L‑03] _grantVotingPower() does note remove old entries before adding new ones

Each time _grantVotingPower() is called, new entries are added to the array, but doing so does not remove any old entries. By calling the function multiple times, an attacker can can increase their voting power indefinitely, without having to acquire new tokens.

```solidity
file:   contracts/NFTBoostVault.sol

521  function _grantVotingPower(address delegatee, uint128 newVotingPower)  internal {
        // update the delegatee's voting power
        History.HistoricalBalances memory votingPower = _votingPower();

        // loads the most recent timestamp of voting power for this delegate
        uint256 delegateeVotes = votingPower.loadTop(delegatee);

        // add block stamp indexed delegation power for this delegate to historical data array
        votingPower.push(delegatee, delegateeVotes + newVotingPower)

```

https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L521-L529

## [L-04] Using zero as a parameter

Taking 0 as a valid argument in Solidity without checks can lead to severe security issues. A historical example is the infamous 0x0 address bug where numerous tokens were lost. This happens because 0 can be interpreted as an uninitialized address, leading to transfers to the 0x0 address, effectively burning tokens. Moreover, 0 as a denominator in division operations would cause a runtime exception. It's also often indicative of a logical error in the caller's code. It's important to always validate input and handle edge cases like 0 appropriately. Use require() statements to enforce conditions and provide clear error messages to facilitate debugging and safer code.

```solidity
file:   contracts/NFTBoostVault.sol
 
153   _registerAndDelegate(user, amount, 0, address(0), delegatee);

171    _lockTokens(msg.sender, uint256(amount), address(0), 0, 0);

286    _lockTokens(msg.sender, amount, address(0), 0, 0);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L153

## [L-05] address shouldn't be hard-coded

It is often better to declare addresses as immutable, and assign them via constructor arguments. This allows the code to remain the same across deployments on different networks, and avoids recompilation when addresses need to change.

```solidity
file:  contracts/ArcadeTreasury.sol

53    address internal constant ETH_CONSTANT = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L53

##  [L‑06] Consider implementing two-step procedure for updating protocol addresses

A copy-paste error or a typo may end up bricking protocol functionality, or sending tokens to an address with no known private key. Consider implementing a two-step procedure for updating protocol addresses, where the recipient is set as pending, and must 'accept' the assignment by making an affirmative call. A straight forward way of doing this would be to have the target contracts implement EIP-165, and to have the 'set' functions ensure that the recipient is of the right interface type.


```solidity
file:    contracts/nft/BadgeDescriptor.sol

57        function setBaseURI(string memory newBaseURI) external onlyOwner {
        baseURI = newBaseURI;

        emit SetBaseURI(msg.sender, newBaseURI);
    }


```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L57


```solidity
file:    contracts/nft/ReputationBadge.sol

184          function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {
        if (_descriptor == address(0)) revert RB_ZeroAddress("descriptor");

        descriptor = IBadgeDescriptor(_descriptor);

        emit SetDescriptor(msg.sender, _descriptor);
    }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L184-L190

```solidity
file:   contracts/token/ArcadeAirdrop.sol

75        function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
        rewardsRoot = _merkleRoot;

        emit SetMerkleRoot(_merkleRoot);
    }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L75-L79

```solidity
file:   contracts/BaseVotingVault.sol

80        function setManager(address manager_) external onlyTimelock {
        if (manager_ == address(0)) revert BVV_ZeroAddress("manager");

        Storage.set(Storage.addressPtr("manager"), manager_);
    }

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L80-L84
