## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#low1-add-to-blacklist-function) | Add to `blacklist` function | 1 |
| [LOW&#x2011;2](#low2-ierc20-approve-is-deprecated) | IERC20 `approve()` Is Deprecated | 1 |
| [LOW&#x2011;3](#low3-the-erc1155sol-contract-does-not-implement-any-security-features-such-as-access-control-or-event-logging) | The `ERC1155.sol` contract does not implement any security features, such as `access control` or `event logging` | 2 |
| [LOW&#x2011;4](#low4-erc20-tokens-may-not-be-compatible-with-significantly-large-approvals) | `ERC20` tokens may not be compatible with significantly large approvals | 1 |
| [LOW&#x2011;5](#low5-functions-calling-contractsaddresses-with-transfer-hooks-are-missing-reentrancy-guards) | Functions calling contracts/addresses with transfer hooks are missing reentrancy guards | 2 |
| [LOW&#x2011;6](#low6-large-transfers-may-not-work-with-some-erc20-tokens) | Large transfers may not work with some `ERC20` tokens | 1 |
| [LOW&#x2011;7](#low7-minting-tokens-to-the-zero-address-should-be-avoided) | Minting tokens to the zero address should be avoided | 1 |
| [LOW&#x2011;8](#low8-contracts-are-not-using-their-oz-upgradeable-counterparts) | Contracts are not using their OZ Upgradeable counterparts | 20 |
| [LOW&#x2011;9](#low9-potential-risks-in-usage-of-erc1155sol) | Potential risks in usage of `ERC1155.sol` | 2 |
| [LOW&#x2011;10](#low10-contracts-are-designed-to-receive-eth-but-do-not-implement-function-for-withdrawal) | Contracts are designed to receive ETH but do not implement function for withdrawal | 1 |
| [LOW&#x2011;11](#low11-unsafe-downcast) | Unsafe downcast | 3 |

Total: 35 contexts over 11 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#nc1-address-shouldnt-be-hardcoded) | `address` shouldn't be hard-coded | 1 |
| [NC&#x2011;2](#nc2-consider-adding-a-denylist) | Consider adding a deny-list | 1 |
| [NC&#x2011;3](#nc3-mint-events-missing-key-information) | Mint events missing key information | 1 |
| [NC&#x2011;4](#nc4-functions-that-alter-state-should-emit-events) | Functions that alter state should emit events | 3 |
| [NC&#x2011;5](#nc5-generate-perfect-code-headers-for-better-solidity-code-layout-and-readability) | Generate perfect code headers for better solidity code layout and readability | 7 |
| [NC&#x2011;6](#nc5-the-nonreentrant-modifier-should-occur-before-all-other-modifiers) | The `nonReentrant` modifier should occur before all other modifiers | 10 |
| [NC&#x2011;7](#nc25-override-function-arguments-that-are-unused-should-have-the-variable-name-removed-or-commented-out-to-avoid-compiler-warnings) | `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings | 2 |
| [NC&#x2011;8](#nc8-it-is-standard-for-all-external-and-public-functions-to-be-overridden-from-an-interface) | It is standard for all external and public functions to be overridden from an interface | 39 |
| [NC&#x2011;9](#nc9-use-smtchecker) | Use SMTChecker | 1 |
| [NC&#x2011;10](#nc10-zero-as-a-function-argument-should-have-a-descriptive-meaning) | Zero as a function argument should have a descriptive meaning | 3 |

Total: 68 contexts over 10 issues

## Low Risk Issues

### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Add to `blacklist` function

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```


#### <ins>Recommended Mitigation Steps</ins>
Add to Blacklist function and modifier.



### <a href="#qa-summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> IERC20 `approve()` Is Deprecated

`approve()` is subject to a known front-running attack. It is deprecated in favor of `safeIncreaseAllowance()` and `safeDecreaseAllowance()`. If only setting the initial allowance to the value that means infinite, `safeIncreaseAllowance()` can be used instead.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#IERC20-approve-address-uint256-

#### <ins>Proof Of Concept</ins>

```solidity
File: ArcadeTreasury.sol

391: IERC20(token).approve(spender, amount);
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L391



#### <ins>Recommended Mitigation Steps</ins>

Consider using `safeIncreaseAllowance()` / `safeDecreaseAllowance()` instead.



### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> The `ERC1155.sol` contract does not implement any security features, such as `access control` or `event logging`

This means that anyone can mint tokens, transfer tokens, and burn tokens.

#### <ins>Proof Of Concept</ins>

```solidity
File: NFTBoostVault.sol

5: import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L5

```solidity
File: ReputationBadge.sol

5: import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L5






### <a href="#qa-summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> `ERC20` tokens may not be compatible with significantly large approvals

Not all `IERC20` implementations are totally compliant, and some (e.g `UNI`, `COMP`) may fail if the valued passed is larger than `uint96`. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)

#### <ins>Proof Of Concept</ins>

```solidity
File: ArcadeTreasury.sol

391: IERC20(token).approve(spender, amount);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L391





### <a href="#qa-summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Functions calling contracts/addresses with transfer hooks are missing reentrancy guards

While adherence to the check-effects-interaction pattern is commendable, the absence of a reentrancy guard in functions, especially where transfer hooks might be present, can expose the protocol users to risks of read-only reentrancies. Such reentrancy vulnerabilities can be exploited to execute malicious actions even without altering the contract state. Without a reentrancy guard, the only potential mitigation would be to blocklist the entire protocol - an extreme and disruptive measure. Therefore, incorporating a reentrancy guard into these functions is vital to bolster security, as it helps protect against both traditional reentrancy attacks and read-only reentrancies, ensuring robust and safe protocol operations.

#### <ins>Proof Of Concept</ins>

```solidity
File: ARCDVestingVault.sol

211: function withdraw(uint256 amount, address recipient) external override onlyManager {
        Storage.Uint256 storage unassigned = _unassigned();
        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
        
        unassigned.data -= amount;

        token.safeTransfer(recipient, amount);
    }

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L211

```solidity
File: ReputationBadge.sol

163: function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {
        if (recipient == address(0)) revert RB_ZeroAddress("recipient");

        
        uint256 balance = address(this).balance;

        
        
        payable(recipient).transfer(balance);

        emit FeesWithdrawn(recipient, balance);
    }

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L163









### <a href="#qa-summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Large transfers may not work with some `ERC20` tokens

Some `IERC20` implementations (e.g `UNI`, `COMP`) may fail if the valued transferred is larger than `uint96`. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)

#### <ins>Proof Of Concept</ins>

```solidity
File: ArcadeTreasury.sol

369: IERC20(token).safeTransfer(destination, amount);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L369






### <a href="#qa-summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Minting tokens to the zero address should be avoided

The core function `mint` is used by users to mint an option position by providing token1 as collateral and borrowing the max amount of liquidity. `Address(0)` check is missing in both this function and the internal function `_mint`, which is triggered to mint the tokens to the to address. Consider applying a check in the function to ensure tokens aren't minted to the zero address.

#### <ins>Proof Of Concept</ins>


```solidity
File: ReputationBadge.sol


function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
        uint256 mintPrice = mintPrices[tokenId] * amount;
        uint48 claimExpiration = claimExpirations[tokenId];

        if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));
        if (msg.value < mintPrice) revert RB_InvalidMintFee(mintPrice, msg.value);
        if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();
        if (amountClaimed[recipient][tokenId] + amount > totalClaimable) {
            revert RB_InvalidClaimAmount(amount, totalClaimable);
        }

        
        amountClaimed[recipient][tokenId] += amount;

        
        _mint(recipient, tokenId, amount, "");
    }

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L98





### <a href="#qa-summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

<details>

```solidity
File: ArcadeTreasury.sol

5: import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
6: import "@openzeppelin/contracts/access/AccessControl.sol";
7: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L5-L7

```solidity
File: ARCDVestingVault.sol

5: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L5

```solidity
File: BaseVotingVault.sol

5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L5

```solidity
File: NFTBoostVault.sol

5: import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";
6: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L5-L6

```solidity
File: BadgeDescriptor.sol

5: import "@openzeppelin/contracts/access/Ownable.sol";
6: import "@openzeppelin/contracts/utils/Strings.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/BadgeDescriptor.sol#L5-L6

```solidity
File: ReputationBadge.sol

5: import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
6: import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Burnable.sol";
7: import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
8: import "@openzeppelin/contracts/access/AccessControl.sol";
9: import "@openzeppelin/contracts/utils/Strings.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L5-L9

```solidity
File: ArcadeAirdrop.sol

5: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeAirdrop.sol#L5

```solidity
File: ArcadeToken.sol

5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
6: import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
7: import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L5-L7

```solidity
File: ArcadeTokenDistributor.sol

5: import "@openzeppelin/contracts/access/Ownable.sol";
6: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L5-L6



</details>

#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts



### <a href="#qa-summary">[LOW&#x2011;9]</a><a name="LOW&#x2011;9"> Potential risks in usage of `ERC1155.sol`

- The contract is not audited, so there is no guarantee that it is secure
- The contract is not interoperable with other ERC1155 contracts, so it cannot be used to exchange tokens with other users

#### <ins>Proof Of Concept</ins>

```solidity
File: NFTBoostVault.sol

5: import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L5

```solidity
File: ReputationBadge.sol

5: import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L5





### <a href="#qa-summary">[LOW&#x2011;10]</a><a name="LOW&#x2011;10"> Contracts are designed to receive ETH but do not implement function for withdrawal

The following contracts can receive ETH but can not withdraw, ETH is occasionally sent by users will be stuck in those contracts.
This functionality also applies to baseTokens resulting in locked tokens and loss of funds.


#### <ins>Proof Of Concept</ins>

```solidity
File: ArcadeTreasury.sol

397: receive() external payable {}
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L397



#### <ins>Recommended Mitigation Steps</ins>
Provide a rescue ETH and rescueTokens function




### <a href="#qa-summary">[LOW&#x2011;11]</a><a name="LOW&#x2011;11"> Unsafe Downcast
When a type is downcast to a smaller type, the higher order bits are truncated, effectively applying a modulo to the original value. Without any other checks, this wrapping will lead to unexpected behavior and bugs

#### <ins>Proof Of Concept</ins>


```solidity
File: ArcadeTreasury.sol

319: lastAllowanceSet[token] = uint48(block.timestamp);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L319

```solidity
File: ARCDVestingVault.sol

106: startTime = uint128(block.number);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L106

```solidity
File: ReputationBadge.sol

108: if (block.timestamp > claimExpiration) revert RB_ClaimingExpired(claimExpiration, uint48(block.timestamp));

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L108




## Non Critical Issues

### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> `address` shouldn't be hard-coded

It is often better to declare `address`es as `immutable`, and assign them via constructor arguments. This allows the code to remain the same across deployments on different networks, and avoids recompilation when addresses need to change.

#### <ins>Proof Of Concept</ins>


```solidity
File: ArcadeTreasury.sol

53: address internal constant ETH_CONSTANT = address(0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L53







### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Consider adding a deny-list

Doing so will significantly increase centralization, but will help to prevent hackers from using stolen tokens

#### <ins>Proof Of Concept</ins>


```solidity
File: ArcadeToken.sol

74: contract ArcadeToken is ERC20, ERC20Burnable, IArcadeToken, ERC20Permit

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L74






### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Mint events missing key information

Some events are missing key information when emitted.
In the following events, they should contain the minted amount and beneficiary to whom it was minted.

#### <ins>Proof Of Concept</ins>


```solidity
File: ArcadeToken.sol

136: emit MinterUpdated(minter);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L136







### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Functions that alter state should emit events

#### <ins>Proof Of Concept</ins>


```solidity
File: BaseVotingVault.sol

68: function setTimelock(address timelock_) external onlyTimelock {
        if (timelock_ == address(0)) revert BVV_ZeroAddress("timelock");

        Storage.set(Storage.addressPtr("timelock"), timelock_);
    }
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L68

```solidity
File: BaseVotingVault.sol

80: function setManager(address manager_) external onlyTimelock {
        if (manager_ == address(0)) revert BVV_ZeroAddress("manager");

        Storage.set(Storage.addressPtr("manager"), manager_);
    }
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L80

```solidity
File: ArcadeTokenDistributor.sol

174: function setToken(IArcadeToken _arcadeToken) external onlyOwner {
        if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");
        if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();

        arcadeToken = _arcadeToken;
    }
}
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L174





### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Generate perfect code headers for better solidity code layout and readability

It is recommended to use pre-made headers for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```


#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: ArcadeTreasury.sol

9: ArcadeTreasury.sol

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L9

```solidity
File: ARCDVestingVault.sol

13: ARCDVestingVault.sol

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L13

```solidity
File: BaseVotingVault.sol

12: BaseVotingVault.sol

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L12

```solidity
File: NFTBoostVault.sol

12: NFTBoostVault.sol

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L12

```solidity
File: BadgeDescriptor.sol

8: BadgeDescriptor.sol

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/BadgeDescriptor.sol#L8

```solidity
File: ReputationBadge.sol

11: ReputationBadge.sol

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L11

```solidity
File: ArcadeToken.sol

9: ArcadeToken.sol

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L9



</details>






### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> The `nonReentrant` modifier should occur before all other modifiers

Currently the `nonReentrant` modifier is not the first to occur, it should occur before all other modifiers.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: ArcadeTreasury.sol

108: function gscSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L108

```solidity
File: ArcadeTreasury.sol

130: function smallSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L130

```solidity
File: ArcadeTreasury.sol

149: function mediumSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L149

```solidity
File: ArcadeTreasury.sol

168: function largeSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L168

```solidity
File: ArcadeTreasury.sol

189: function gscApprove(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L189

```solidity
File: ArcadeTreasury.sol

211: function approveSmallSpend(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L211

```solidity
File: ArcadeTreasury.sol

230: function approveMediumSpend(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L230

```solidity
File: ArcadeTreasury.sol

249: function approveLargeSpend(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L249

```solidity
File: ArcadeTreasury.sol

333: function batchCalls(
        address[] memory targets,
        bytes[] calldata calldatas
    ) external onlyRole(ADMIN_ROLE) nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L333

```solidity
File: NFTBoostVault.sol

139: function airdropReceive(
        address user,
        uint128 amount,
        address delegatee
    ) external override onlyAirdrop nonReentrant {
```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L139



</details>

#### <ins>Recommended Mitigation Steps</ins>

Re-sort modifiers so that the `nonReentrant` modifier occurs first.




### <a href="#qa-summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings

#### <ins>Proof Of Concept</ins>


```solidity
File: BaseVotingVault.sol

96: function queryVotePower : bytes calldata

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L96

```solidity
File: ImmutableVestingVault.sol

40: function revokeGrant : address

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ImmutableVestingVault.sol#L40




### <a href="#qa-summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> It is standard for all external and public functions to be overridden from an interface

Check that all public or external functions are override. This is iseful to make sure that the whole API is extracted in an interface.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: ArcadeTreasury.sol

108: function gscSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L108

```solidity
File: ArcadeTreasury.sol

130: function smallSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L130

```solidity
File: ArcadeTreasury.sol

149: function mediumSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L149

```solidity
File: ArcadeTreasury.sol

168: function largeSpend(
        address token,
        uint256 amount,
        address destination
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L168

```solidity
File: ArcadeTreasury.sol

189: function gscApprove(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(GSC_CORE_VOTING_ROLE) nonReentrant {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L189

```solidity
File: ArcadeTreasury.sol

211: function approveSmallSpend(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L211

```solidity
File: ArcadeTreasury.sol

230: function approveMediumSpend(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L230

```solidity
File: ArcadeTreasury.sol

249: function approveLargeSpend(
        address token,
        address spender,
        uint256 amount
    ) external onlyRole(CORE_VOTING_ROLE) nonReentrant {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L249

```solidity
File: ArcadeTreasury.sol

269: function setThreshold(address token, SpendThreshold memory thresholds) external onlyRole(ADMIN_ROLE) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L269

```solidity
File: ArcadeTreasury.sol

303: function setGSCAllowance(address token, uint256 newAllowance) external onlyRole(ADMIN_ROLE) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L303

```solidity
File: ArcadeTreasury.sol

333: function batchCalls(
        address[] memory targets,
        bytes[] calldata calldatas
    ) external onlyRole(ADMIN_ROLE) nonReentrant {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ArcadeTreasury.sol#L333

```solidity
File: ARCDVestingVault.sol

91: function addGrantAndDelegate(
        address who,
        uint128 amount,
        uint128 cliffAmount,
        uint128 startTime,
        uint128 expiration,
        uint128 cliff,
        address delegatee
    ) external onlyManager {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L91

```solidity
File: ARCDVestingVault.sol

157: function revokeGrant(address who) external virtual onlyManager {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L157

```solidity
File: ARCDVestingVault.sol

197: function deposit(uint256 amount) external onlyManager {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L197

```solidity
File: ARCDVestingVault.sol

260: function delegate(address to) external {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L260

```solidity
File: ARCDVestingVault.sol

293: function claimable(address who) external view returns (uint256) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L293

```solidity
File: ARCDVestingVault.sol

304: function getGrant(address who) external view returns (ARCDVestingVaultStorage.Grant memory) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/ARCDVestingVault.sol#L304

```solidity
File: BaseVotingVault.sol

68: function setTimelock(address timelock_) external onlyTimelock {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L68

```solidity
File: BaseVotingVault.sol

80: function setManager(address manager_) external onlyTimelock {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L80

```solidity
File: BaseVotingVault.sol

112: function queryVotePowerView(address user, uint256 blockNumber) external view returns (uint256) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L112

```solidity
File: BaseVotingVault.sol

126: function timelock() public view returns (address) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L126

```solidity
File: BaseVotingVault.sol

137: function manager() public view returns (address) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/BaseVotingVault.sol#L137

```solidity
File: NFTBoostVault.sol

697: function onERC1155Received(address, address, uint256, uint256, bytes memory) public virtual returns (bytes4) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L697

```solidity
File: BadgeDescriptor.sol

57: function setBaseURI(string memory newBaseURI) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/BadgeDescriptor.sol#L57

```solidity
File: ReputationBadge.sol

98: function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L98

```solidity
File: ReputationBadge.sol

140: function publishRoots(ClaimData[] calldata _claimData) external onlyRole(BADGE_MANAGER_ROLE) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L140

```solidity
File: ReputationBadge.sol

163: function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L163

```solidity
File: ReputationBadge.sol

184: function setDescriptor(address _descriptor) external onlyRole(RESOURCE_MANAGER_ROLE) {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/nft/ReputationBadge.sol#L184

```solidity
File: ArcadeAirdrop.sol

62: function reclaim(address destination) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeAirdrop.sol#L62

```solidity
File: ArcadeAirdrop.sol

75: function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeAirdrop.sol#L75

```solidity
File: ArcadeToken.sol

132: function setMinter(address _newMinter) external onlyMinter {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L132

```solidity
File: ArcadeToken.sol

145: function mint(address _to, uint256 _amount) external onlyMinter {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeToken.sol#L145

```solidity
File: ArcadeTokenDistributor.sol

73: function toTreasury(address _treasury) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L73

```solidity
File: ArcadeTokenDistributor.sol

89: function toDevPartner(address _devPartner) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L89

```solidity
File: ArcadeTokenDistributor.sol

105: function toCommunityRewards(address _communityRewards) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L105

```solidity
File: ArcadeTokenDistributor.sol

121: function toCommunityAirdrop(address _communityAirdrop) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L121

```solidity
File: ArcadeTokenDistributor.sol

138: function toTeamVesting(address _vestingTeam) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L138

```solidity
File: ArcadeTokenDistributor.sol

155: function toPartnerVesting(address _vestingPartner) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L155

```solidity
File: ArcadeTokenDistributor.sol

174: function setToken(IArcadeToken _arcadeToken) external onlyOwner {

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/token/ArcadeTokenDistributor.sol#L174



</details>





### <a href="#qa-summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19




### <a href="#qa-summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Zero as a function argument should have a descriptive meaning

Consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.

#### <ins>Proof Of Concept</ins>


```solidity
File: NFTBoostVault.sol

153: _registerAndDelegate(user, amount, 0, address(0), delegatee);
171: _lockTokens(msg.sender, uint256(amount), address(0), 0, 0);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L153

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L171



```solidity
File: NFTBoostVault.sol

286: _lockTokens(msg.sender, amount, address(0), 0, 0);

```

https://github.com/code-423n4/2023-07-arcade/tree/main/contracts/NFTBoostVault.sol#L286






