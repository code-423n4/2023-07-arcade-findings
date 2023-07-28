### 1. Approach taken in evaluating the codebase
- Looked through the code to get a sensing of what the protocol intends to do
- Once I get the big picture (voting power, NFT boost, ERC20-token as votes), I start to focus on the core contracts like NFTBoostVault and ARCDVestingVault.
- Checked all the calculation thoroughly to see whether there is any miscalculation or any way the users can abuse the voting system
- Think of some scenarios that may happen during NFTBoostVault and whether all the edge cases have been worked out and all the votes tallies correctly

1. Deposit with nft
2. Deposit without nft
3. Deposit without nft and deposit nft halfway
4. Deposit with nft and withdraw nft halfway
5. Deposit nft, withdraw nft, deposit tokens, deposit nft, withdraw tokens

### 2. Codebase quality analysis

The codebase is very well written. Simple idea, but executed well. Usage of storage pointer is also neat. The creation of the ArcadeToken is very well organized. Imports such as ERC20Burnable, ERC20 permit is used correctly. ERC1155 NFT creation is well created. Only small thing is that the msg.value should == mintPrice when minting an NFT, if not people may overpay for the ERC1155 token.  Overall well written code and easy to understand.

### 3. Mechanism Review

Let's look at ARCDVestingVault
1. Grant recipient gets grant amount
2. Grant amount can only be withdrawn after the cliff
3. All the grant amount can be withdrawn after expiration
4. The grant amount is used as voting power in line 146.
5. When grant amount is withdrawn, the voting power decreases proportionately

I was thinking that if the grant recipient does not have access to the grant amount, it shouldn't be counted as voting power. In other words, if the grant amount is not liquid but locked under time, the grant recipient should not have all the voting power. For example, if I have a 1 million grant but it starts to unlock in a decade (cliff set a decade later), I should only have voting power once I can start claiming. Otherwise those that have a huge grant but is under timelock (before cliff) have lots of unrealized voting power. Maybe the voting power should only be vested once the cliff is reached, but that's my opinion.

NFTBoostVault
Voting power is proportionate to amount of Arcade tokens deposited, and users can increase their proportion of votes by up to 150% by staking an ERC1155 NFT. The whole protocol revolves around itself, so no other ERC1155 NFT can be used except Arcade, and no other tokens can be deposited for voting power except the arcade tokens. ERC1155 NFTs that are used as boost will be locked inside the NFTBoostVault contract. The user can withdraw their NFT anytime, and will have their voting power updated accordingly. Seems like everything work in tandem.

### 4. Architecture Review

ARCDVestingVault.sol

The input parameter of startTime, expiration and cliff is a bit confusing because it is unknown whether these three timing variables uses block.number or block.timestamp. Inside the comments, it is mentioned that expiration and cliff are using timestamp, 

```
     * @param startTime                  Optionally set a start time in the future. If set to zero
     *                                   then the start time will be made the block tx is in.
     * @param expiration                 Timestamp when the grant ends - all tokens are unlocked and withdrawable.
     * @param cliff                      Timestamp when the cliff ends. cliffAmount tokens are withdrawable when
```

but startTime uses block.number. I think the protocol intends to use block.number for all 3 variables because of the code in `claim()`, so it would be nice if the comments are synced together and all variables use block.number. 

ArcadeToken.sol

The self-imposed restrictions on minting Arcade Token is a nice touch. 100 million tokens with a 2% increase in totalSupply every 365 days make it so that the minter cannot do unlimited mints, which adds a layer of protection against centralization risk.

### 5. Centralization risks

There is a lot of power placed on the admins, such as the manager role. For example, a manager can revoke any grant at any time and withdraw the grant amount, which is extremely dangerous. The core voting role also controls the treasury, and they could empty the treasury at any time. 

### Time spent:
12 hours