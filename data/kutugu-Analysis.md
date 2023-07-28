# Introduction

Arcade is a platform to enable liquid lending markets for NFTs. The eepository for this audit contains the contracts for
a token-based governance system, which can be granted ownership and management authority over the core lending protocol.

# Observations

## Systemic risks

### Dependency

```shell
    "@openzeppelin/contracts": "4.3.2",
    "@openzeppelin/contracts-upgradeable": "^4.7.2",
```

The contract relies on a low version of openzeppelin with some high-risk vulnerabilities, which leaves some attack
vectors.

### Type conversion

The contract has unsafe type conversions, and although overflow is unlikely, it is better to use safecast for
conversions.

## Mechanism review

### delegate

```solidity
// @ARCDVestingVault
function addGrantAndDelegate() {
    if (grant.allocation != 0) revert AVV_HasGrant();
}

function delegate(address to) external {
    grant.delegatee = to;
}

// @NFTBoostVault
function _registerAndDelegate() {
    if (registration.delegatee != address(0)) revert NBV_HasRegistration();
}

function delegate(address to) external override {
    if (to == address(0)) revert NBV_ZeroAddress("to");
}

```

The ARCDVestingVault and NFTBoostVault are designed in different ways to defend against multiple times registe attacks:
-   ARCDVestingVault check `allocation != 0` to defend registe multiple times and allow update delegate to `address(0)`
-   NFTBoostVault check `delegatee != address(0)` to defend registe multiple times and don't allow update delegate to
    `address(0)`

While this does not present a security issue, different mechanism designs may make it difficult to understand the system
and future upgrades may create incompatibility issues. It is better to unify the two into one defensive strategy.

### ARCDVestingVault

#### VotingPower

There is a `cliff` cooling period in ARCDVestingVault, but votePower is not affected by this.

### NFTBoostVault

#### latestVotingPower

In NFTBoostVault, since the NFT boost multipler is variable, it is important to note that `latestVotingPower` is old
data, and getting the current new data should always be recalculated using `_currentVotingPower`, which is handled well.

#### getMultiplier

The `getMultiplier` function implementation has problems, when reading multipliers of non-boost NFT, getMultiplier
should return `1e3`, not `0`. This combined with the `updateNft` issue, caused users to temporarily lose votePower.

#### updateNft

The updateNft did not verify that the address and Id passed in by the user were of boost type; if the user passed in
non-boost NFT, the getMultiplier returned 0, resulting in the user losing voting rights.

### ArcadeTokenDistributor

Contracts have some factors to consider and are handled well in the code:

-   The token allocation needs to be guaranteed 100% and there will be no token left in the contract
-   Use flag to ensure that it can be extracted only once

### NFT

#### tokenId

NFTBoostVault does not support token whose tokenId is 0, which makes the protocol incompatible with the type of
`tokenId == 0`. However, the ERC1155 standard does not specify that the tokenId cannot be 0.

#### safeTransferFrom

The safeTransferFrom helps prevent NFT from getting stuck in unsupported contracts, but this introduces a vector of
reentrant attacks.  
While the contract correctly adds reentrant protections for executable functions, but there is no protection for
readability functions such as `getRegistration`.    
When the user withdraws the token, votePower remains unchanged, other contracts that rely on these view functions should
be careful.    

#### onERC1155Received

The contract should avoid accepting non-boost NFT, which may cause the NFT to accidentally send and stuck in the
contract.

#### tokenURI

According to the ERC721 standard, tokenURI should revert when the tokenId does not exist to prevent phishing attacks
from deceiving the user. However, the contract will return the corresponding link, which does not comply with the
specification.


### Time spent:
15 hours