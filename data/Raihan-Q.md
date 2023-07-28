# low severity

# SUMMARY 

|      |   issue   | instance |
|------|-----------|----------|
|[G-01]| Use of abi.encodePacked with dynamic types inside keccak256|1|
|[G-02]| Contracts are not using their OZ upgradeable counterparts|18|
|[G-03]| SetBaseURI emits wrong caller address|2|
|[G-04]| Replicate checks from constructor in queryVotePower functions|1|
|[G-05]| Low-level calls that are unnecessary for the system should be avoided|1|
|[G-06]| Don’t use payable.transfer()/payable.send()|2|
|[G-07]| Missing event and or timelock for critical parameter change|1|
|[G-08]| NFT doesn't handle hard forks|1|
|[G-09]| Unbounded loop|7|
|[G-10]|  mint function in the ReputationBadge contract does not perform any input validation|1|
|[G-11]| The withdraw function in the provided code does not perform any access control or permission |1|


# Details

## [L-01] Use of abi.encodePacked with dynamic types inside keccak256
abi.encodePacked should not be used with dynamic types when passing the result to a hash function such as keccak256. Use abi.encode instead, which will pad items to 32 bytes, to [prevent any hash collisions](https://docs.soliditylang.org/en/latest/abi-spec.html#non-standard-packed-mode).

```solidity
File: contracts/nft/ReputationBadge.sol
211        bytes32 leafHash = keccak256(abi.encodePacked(recipient, tokenId, totalClaimable));
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L211

## [L-02] Contracts are not using their OZ upgradeable counterparts
Description
The non-upgradeable standard version of OpenZeppelinâ€™s library, such as `Ownable`, `Pausable`, `Address`, `Context`, `SafeERC20`, `ERC1967Upgrade` etc, are inherited / used by both the proxy and the implementation contracts.

As a result, when attempting to use the upgrades plugin mentioned, the following errors are raised:

```
Error: Contract `FiatTokenV1` is not upgrade safe

contracts/v1/FiatTokenV1.sol:58: Variable `totalSupply_` is assigned an initial value
  Move the assignment to the initializer
  https://zpl.in/upgrades/error-004

contracts/v1/Pausable.sol:49: Variable `paused` is assigned an initial value
  Move the assignment to the initializer
  https://zpl.in/upgrades/error-004

contracts/v1/Ownable.sol:28: Contract `Ownable` has a constructor
  Define an initializer instead
  https://zpl.in/upgrades/error-001

contracts/util/Address.sol:186: Use of delegatecall is not allowed
  https://zpl.in/upgrades/error-002
       
```
Having reviewed these errors, none had any adversarial impact:

`totalSupply_` and `paused` are explictly assigned the default values `0` and `false`
the implementation contracts utilises the internal `_transferOwnership()` in the initializer, thus transferring ownership to `newOwner` regardless of who the current owner is
`Address's` `delegatecall` is only used by the `ERC1967Upgrade` contract. Comparing both the `Address` and `ERC1967Upgrade` contracts against their upgradeable counterparts show similar behaviour (differences are some refactoring done to shift the delegatecall into the `ERC1967Upgrade` contract).
Nevertheless, it would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

```solidity
File: contracts/ArcadeTreasury.sol
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L5-L7

```solidity
File: contracts/ARCDVestingVault.sol
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L5

```solidity
File: contracts/BaseVotingVault.sol
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L5

```solidity
File: contracts/NFTBoostVault.sol
import "@openzeppelin/contracts/token/ERC1155/IERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L5-L6

```solidity
File: contracts/nft/BadgeDescriptor.sol
import "@openzeppelin/contracts/access/Ownable.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L5

```solidity
File: contracts/nft/ReputationBadge.sol
import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/token/ERC1155/extensions/ERC1155Burnable.sol";
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L5-L8

```solidity
File: contracts/token/ArcadeAirdrop.sol
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol#L5

```solidity
File: contracts/token/ArcadeToken.sol
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol#L5-L7

```solidity
File: contracts/token/ArcadeTokenDistributor.sol
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L5-L6

### Recommended Mitigation Steps
Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.

## [L-03] SetBaseURI emits wrong caller address
In the BadgeDescriptor.SetBaseURI function the SetBaseURI is emitted ([Link](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L60)).

It is defined as:
[Link](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L20)

```solidity
File: contracts/nft/BadgeDescriptor.sol
20    event SetBaseURI(address indexed caller, string baseURI);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L20

So the first parameter should be the caller address.

When the event is emitted the first parameter is msg.sender. The issue is that the caller address and msg.sender can be different. So the event in some cases contains wrong information.

### Fix:
```
-   emit SetBaseURI(msg.sender, newBaseURI);
+   emit SetBaseURI(caller, newBaseURI);
```


Another Same issue here
```solidity
File: contracts/nft/ReputationBadge.sol
61    event SetDescriptor(address indexed caller, address indexed descriptor);

189   emit SetDescriptor(msg.sender, _descriptor);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L61

### Fix:
```solidity
-  emit SetDescriptor(msg.sender, _descriptor);
+  emit SetDescriptor(caller, _descriptor);
```

## [L-04] Replicate checks from constructor in queryVotePower functions
In the BaseVotingVault [constructor](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L52-L58 it is checked that _staleBlockLag >= block.number ([Link](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L54)).

These checks are not implemented in the [queryVotePower](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/BaseVotingVault.sol#L112-L118) functions.

It is recommended to add the check to queryVotePower functions such that it is ensured the functions do not cause _staleBlockLag and block.number to be set to bad values.

### Fix:
```solidity
 function queryVotePower(address user, uint256 blockNumber, bytes calldata) external override returns (uint256) {
    // Check that staleBlockLag is not greater than the current block number
    require(staleBlockLag < block.number, "staleBlockLag too high");

    // Get our reference to historical data
    History.HistoricalBalances memory votingPower = _votingPower();

    // Find the historical data and clear everything more than 'staleBlockLag' into the past
    return votingPower.findAndClear(user, blockNumber, block.number - staleBlockLag);
}
```


## [L-05] Low-level calls that are unnecessary for the system should be avoided
Low-level calls that are unnecessary for the system should be avoided whenever possible because low-level calls behave differently from a contract-type call.
```solidity
File: contracts/ArcadeTreasury.sol
341    (bool success, ) = targets[i].call(calldatas[i]);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L341


## [L‑06] Don’t use payable.transfer()/payable.send()
The use of payable.transfer() is heavily frowned upon because it can lead to the locking of funds. The transfer() call requires that the recipient is either an EOA account, or is a contract that has a payable callback. For the contract case, the transfer() call only provides 2300 gas for the contract to complete its operations. This means the following cases can cause the transfer to fail:
- The contract does not have a payable callback
- The contract’s payable callback spends more than 2300 gas (which is only enough to emit something)
- The contract is called through a proxy which itself uses up the 2300 gas Use OpenZeppelin’s Address.sendValue() instead
[Reffrence](https://code4rena.com/reports/2022-07-golom#l02--dont-use-payabletransferpayablesend)

```solidity
File: contracts/ArcadeTreasury.sol
367       payable(destination).transfer(amount);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol#L367

```solidity
File: contracts/nft/ReputationBadge.sol
171         payable(recipient).transfer(balance);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L171

## [L‑07] Missing event and or timelock for critical parameter change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
```solidity
File: contracts/token/ArcadeTokenDistributor.sol
function setToken(IArcadeToken _arcadeToken) external onlyOwner {
        if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");
        if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();

        arcadeToken = _arcadeToken;
    }
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol#L174-L179

```solidity
File: 
```
https://github.com/code-423n4/2023-07-arcade/blob/main/#L


## [L‑08] Protect LlamaPolicy.sol NFT from copying in POW forks
Ethereum has performed the long-awaited “merge” that will dramatically reduce the environmental impact of the network

There may be forked versions of Ethereum, which could cause confusion and lead to scams as duplicated NFT assets enter the market.

If the Ethereum Merge, which took place in September 2022, results in the Blockchain splitting into two Blockchains due to the ‘THE DAO’ attack in 2016, this could result in duplication of immutable tokens (NFTs).

In any case, duplicate NFTs will exist due to the ETH proof-of-work chain and other potential forks, and there’s likely to be some level of confusion around which assets are ‘official’ or ‘authentic.’

Even so, there could be a frenzy for these copies, as NFT owners attempt to flip the proof-of-work versions of their valuable tokens.

As ETHPOW and any other forks spin off of the Ethereum mainnet, they will yield duplicate versions of Ethereum’s NFTs. An NFT is simply a blockchain token, and it can work as a deed of ownership to digital items like artwork and collectibles. A forked Ethereum chain will thus have duplicated deeds that point to the same tokenURI.

About Merge Replay Attack: https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

[Reffrence](https://code4rena.com/reports/2023-06-llama#l-03-protect-llamapolicysol--nft-from-copying-in-pow-forks)

```solidity
File: contracts/nft/BadgeDescriptor.sol
48    function tokenURI(uint256 tokenId) external view override returns (string memory) {
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/BadgeDescriptor.sol#L48

Recommended Mitigation Steps
Add the following check:

```
if(block.chainid != 1) { 
    revert(); 
}
```

## [L-09] Unbounded loop
New items are pushed into the following arrays but there is no option to pop them out. Currently, the array can grow indefinitely. E.g. there’s no maximum limit and there’s no functionality to remove array values.

If the array grows too large, calling relevant functions might run out of gas and revert. Calling these functions could result in a DOS condition.

```solidity
File: contracts/ARCDVestingVault.sol
146        votingPower.push(grant.delegatee, delegateeVotes + newVotingPower);

268        votingPower.push(grant.delegatee, oldDelegateeVotes - grant.latestVotingPower);

276        votingPower.push(to, newDelegateeVotes + grant.latestVotingPower);

352        votingPower.push(grant.delegatee, delegateeVotes - uint256(change * -1));

```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L146

```solidity
File: contracts/NFTBoostVault.sol
194        votingPower.push(registration.delegatee, oldDelegateeVotes - registration.latestVotingPower);

205        votingPower.push(to, newDelegateeVotes + addedVotingPower);

529        votingPower.push(delegatee, delegateeVotes + newVotingPower);
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/NFTBoostVault.sol#L194

Recommended Mitigation Steps
Add a functionality to delete array values or add a maximum size limit for arrays.

## [L-10] mint function in the ReputationBadge contract does not perform any input validation
The mint function in the provided code does not perform any input validation on the recipient, tokenId, amount, totalClaimable, or merkleProof parameters. This leaves the function open to potential vulnerabilities such as integer overflow or underflow that could lead to unexpected behavior or even allow for unauthorized minting.

```solidity
File: contracts/nft/ReputationBadge.sol
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

        // increment amount claimed
        amountClaimed[recipient][tokenId] += amount;

        // mint to recipient
        _mint(recipient, tokenId, amount, "");
    }
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol#L98-L120

## [G-11] The withdraw function in the provided code does not perform any access control or permission 
The withdraw function in the provided code does not perform any access control or permission checks to ensure that only authorized users are able to call it. This leaves the function open to abuse by malicious actors who may attempt to withdraw funds without proper authorization.

```solidity
File: contracts/ARCDVestingVault.sol
  function withdraw(uint256 amount, address recipient) external override onlyManager {
        Storage.Uint256 storage unassigned = _unassigned();
        if (unassigned.data < amount) revert AVV_InsufficientBalance(unassigned.data);
        // update unassigned value
        unassigned.data -= amount;

        token.safeTransfer(recipient, amount);
    }
```
https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ARCDVestingVault.sol#L211-L218