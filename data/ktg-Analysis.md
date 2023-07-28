### Arcade audit's contracts analysis

The followings are my understanding of Arcade audit main contracts

- Vault contracts:
    - `BaseVotingVault`: The contract provides basic voting vault functions,
  it tracks and allows querying users voting power. All 3 vaults `ARCDVestingVault`, `NFTBoostVault`
  and `ImmutableVestingVault` inherits this contract

    - `ARCDVestingVault`: This is a vesting vault for Arcade token with the following main
    features
        +   The manager deposits arcade tokens into the vault, he then
            creates grants for users
        +   A grant is an amount of money that assigned users can claim.
            User can only claim grant after cliff time, the amount to claim
            if the cliff amount plus a linear time dependent amount until all allocation
            in grant is withdrawn.
        +   A grant also include a delegatee. This delegatee will receive
            voting power equals total allocation - withdrawn amount.
        +   The manager can revoke a grant, if that happens, the withdrawable
            amount (cliff amount plus linear time dependent amount) will be transferred
            to the grant owner, the remaining goes to the manager.
    - `ImmutableVestingVault`: This contract is exactly the same with `ARCDVestingVault`,
        except that revoking grant is disabled.
    - `NFTBoostVault`: This contract contains the most logic in Arcade audit, its main
    features are as follows:
        +   Users *register* by depositing ERC20 tokens into the vault and receive
            voting power. They can also optionally provide a nft token to use as *multiplier*, which
            multiplies their voting power
        +   Only manager can set multiplier for tokens
        +   Users can add more tokens to their registrations or update
            the multiplier nft associated with them.
        +   If user choose to *withdraw* their funds, their voting
            power is reduced accordingly. Withdrawing feature is only
            enabled by time lock contract by calling *unlock* function


- Token contracts:
    - `ArcadeToken`: an ERC20 token with main features:
        +   Only *minter* can mint new tokens, *minter* is set
            in the constructor and then only this *minter* can set new minter
        +   An inflationary cap of 2% per year is enforced on each mint
    - `ArcadeTokenDistributor`: This is the contract for distributing arcade tokens,
    it contains predefined amounts of tokens that needed to be transferred to specific
    organization/user and also flags indicating if those amounts are sent. After a
    specific amount is already sent, it cannot be sent again.
    - `ArcadeAirdrop`: This contract receives arcade tokens from `ArcadeTokenDistributor` and
    allows users to claim it using merkle proofs.
- Badge related contracts:
    - `BadgeDescriptor`: This is to be used with `ReputationBadge` to allow
    querying token URI
    - `ReputationBadge`: This is an ERC1155 contract, its tokens are used
        as `multiplier` in `NFTBoostVault`. User must pay ETH to mint new
        tokens, and then they can use these tokens as multipliers.



### Arcade main contracts risks analysis:
Following is my risk analysis of Arcade main contracts, namely `ARCDVestingVault`
and `NFTBoostVault`, which contains the most logic.
- `ARCDVestingVault`:
    +  About access control and centralization related problems, this contract gives the manager also all privileges: only manager
        can create grant, deposit the fund, withdrawing tokens... they can also revoke any grant
        they want regardless of the status of the grant. Users can just receive grants
        and delegate the voting power associated with them.
    + Since the users can't do much and the manager is trusted, there can be hardly
     any change anyone can do something they're not supposed to.
    + The contract does an excellent job in calculating token amount and
     voting power when grants are created, delegatee changed and other situations.
    + At the end, I found no problem with this contract
- `NFTBoostVault`:
    +   This contract potentially contains more problems since it allows user to conduct
        more actions compare to `ARCDVestingVault`. User can deposit tokens and nft to vaults,
        add tokens to their registrations, update their nfts or event withdraw the nft.
    +   The main problems I found with this contract is how to keep voting power
        updated and also its compatibility with `ReputationBadge` contract:
        +   The manager can update a token's multiplier by calling `setMultiplier` 
            but it will not update voting power of users using those tokens
        +   The contract does not allow token id = 0, but ERC1155 token id = 0
           is perfectly normal and also `ReputationBadge` supports this case
        +  The contract uses *uint128* to process token id, but  `ReputationBadge` contract
            use *uint256*
    +   There is no external/public function to retrieve the exact current
        voting power of a user, *_currentVotingPower* in internal
    +   One thing quite strange with me is the fact that *withdraw* feature is only
        enabled by the manager, and once it's enabled, it cannot be reversed. I don't know
        the story behind this but I think it sends a mixed message: the manager
        is trusted enough to be granted power to enable withdraw, but then not allowed
        to reverse it. I think this problem should be considered, if the manager is trusted,
        then he should be allowed to pause withdraw in critical cases.
  




### Time spent:
20 hours