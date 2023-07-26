### Token transfer

**contracts/nft/ReputationBadge.sol#L171**

Used "transfer" to withdraw/transfer the funds from contract. Must preferred .call() to transfer the funds


```Solidity
function withdrawFees(address recipient) external onlyRole(BADGE_MANAGER_ROLE) {
        if (recipient == address(0)) revert RB_ZeroAddress("recipient");

        // get contract balance
        uint256 balance = address(this).balance;

        // transfer balance to recipient
        // will out-of-gas revert if recipient is a contract with logic inside receive()
        //payable(recipient).transfer(balance); // @audit Should use call() function instead of transfer
        
        (bool sent, ) = payable(recipient).call{value: balance}("");
        require(sent, "Failed to send Ether");

        emit FeesWithdrawn(recipient, balance);
    }
```



### Gas optimisation possibilities.

**contracts/NFTBoostVault.sol#L627**
Should be preferred to return uint128 instead uint256

```Solidity
    function _currentVotingPower(
        NFTBoostVaultStorage.Registration memory registration
    ) internal view virtual returns (uint128) {
        uint128 locked = registration.amount - registration.withdrawn;

        if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
            return (locked * getMultiplier(registration.tokenAddress, registration.tokenId)) / MULTIPLIER_DENOMINATOR;
        }

        return locked;
    }
```


**contracts/token/ArcadeToken.sol#L80**
Should be preferred as uint32
```Solidity
    //uint48 public constant MIN_TIME_BETWEEN_MINTS = 365 days;
    uint32 public constant MIN_TIME_BETWEEN_MINTS = 365 days;
```

**contracts/token/ArcadeToken.sol#L83**
Should be preferred as uint8

```Solidity
    // uint256 public constant MINT_CAP = 2;
    uint8 public constant MINT_CAP = 2;
```

**contracts/token/ArcadeToken.sol#L86**
Should be preferred as uint8
```Solidity
    // uint256 public constant PERCENT_DENOMINATOR = 100;
    // uint8 public constant PERCENT_DENOMINATOR = 100;
```

**contracts/token/ArcadeToken.sol#L89**
Should be preferred as uint128
```Solidity
    // uint256 public constant INITIAL_MINT_AMOUNT = 100_000_000 ether;
    uint128 public constant INITIAL_MINT_AMOUNT = 100_000_000 ether;
```

**contracts/token/ArcadeTokenDistributor.sol#L32 - #L57**
uin256 constants should be preferred as uint128

```Solidity
   // uint256 public constant treasuryAmount = 25_500_000 ether;
   uint128 public constant treasuryAmount = 25_500_000 ether;
    
   //  uint256 public constant devPartnerAmount = 600_000 ether;
    uint256 public constant devPartnerAmount = 600_000 ether;

   // uint256 public constant communityRewardsAmount = 15_000_000 ether;
    uint128 public constant communityRewardsAmount = 15_000_000 ether;

    // uint256 public constant communityAirdropAmount = 10_000_000 ether;
    uint128 public constant communityAirdropAmount = 10_000_000 ether;

    // uint256 public constant vestingPartnerAmount = 32_700_000 ether;
   uint128 public constant vestingPartnerAmount = 32_700_000 ether;

```

**contracts/external/council/CoreVoting.sol#L15 - #L24**
uin256 constants should be preferred as uint16

```Solidity
    // uint256 public constant DAY_IN_BLOCKS = 6496;
    uint16 public constant DAY_IN_BLOCKS = 6496;

    // uint256 public lockDuration = DAY_IN_BLOCKS * 3;
    uint16 public lockDuration = DAY_IN_BLOCKS * 3;

   // uint256 public extraVoteTime = DAY_IN_BLOCKS * 5;
    uint16 public extraVoteTime = DAY_IN_BLOCKS * 5;
```
