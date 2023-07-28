# Low 



|      |  ISSUE  |  INSTANCE  |
|------|---------|------------|
|[L-01]|Contracts are not using their OZ upgradeable counterparts|11|
|[L-02]|Missing event and or timelock for critical parameter change|1|
|[L-03]|Use of abi.encodePacked with dynamic types inside keccak256|1|
|[L-04]|Low-level calls that are unnecessary for the system should be avoided|1|
|[L-05]|SetBaseURI emits wrong caller address|1|

## [L-01] Contracts are not using their OZ upgradeable counterparts
Description
The non-upgradeable standard version of OpenZeppelinâ€™s library, such as `Ownable`, `Pausable`, `Address`, `Context`, `SafeERC20`, `ERC1967Upgrade` etc, are inherited / used by both the proxy and the implementation contracts.

```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L5

```solidity
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L5

```solidity
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L5

```solidity
import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L5

```solidity
import "@openzeppelin/contracts/access/Ownable.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L5

```solidity
import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Burnable.sol";
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L5

```solidity
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L5

```solidity
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L5

## [L‑02] Missing event and or timelock for critical parameter change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
```solidity
function setToken(IArcadeToken _arcadeToken) external onlyOwner {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L174

## [L-03] Use of abi.encodePacked with dynamic types inside keccak256
abi.encodePacked should not be used with dynamic types when passing the result to a hash function such as keccak256. Use abi.encode instead, which will pad items to 32 bytes, to prevent any hash collisions.

```solidity
bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L211


## [L-04] Low-level calls that are unnecessary for the system should be avoided
Low-level calls that are unnecessary for the system should be avoided whenever possible because low-level calls behave differently from a contract-type call.

```solidity
341    (bool success, ) = targets[i].call(calldatas[i]);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341

## [L-05] SetBaseURI emits wrong caller address
In the BadgeDescriptor.SetBaseURI function the SetBaseURI is emitted

```solidity
20    event SetBaseURI(address indexed caller, string baseURI);
60    emit SetBaseURI(msg.sender, newBaseURI);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L20

Fix:
```
20    event SetBaseURI(address indexed caller, string baseURI);
60    emit SetBaseURI(caller, newBaseURI);
``` 