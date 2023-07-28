### [L-01] `_staleBlockLag` should be greater than 0 in `BaseVotingVault`
If `_staleBlockLag == 0`, all users' voting power will be 0 because it gets the voting power at `block.number` and we use the voting power for past blocks only due to the [flashloan attack](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/external/council/CoreVoting.sol#L175).

- https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/BaseVotingVault.sol#L57

```solidity
    constructor(IERC20 _token, uint256 _staleBlockLag) {
        if (address(_token) == address(0)) revert BVV_ZeroAddress("token");
        if (_staleBlockLag >= block.number) revert BVV_UpperLimitBlock(_staleBlockLag);

        token = _token;
        staleBlockLag = _staleBlockLag;
    }
```

### [L-02] `delegate()` works with the empty registration

Users can delegate with the empty registration and they can bypass the limitation in [updateNft()](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/NFTBoostVault.sol#L314) and [addTokens()](https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/NFTBoostVault.sol#L273).

- https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/NFTBoostVault.sol#L182

```solidity
    function delegate(address to) external override {
```

- https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/ARCDVestingVault.sol#L260

```solidity
    function delegate(address to) external {
```

### [L-03] Users wouldn't withdraw all tokens when it's impossible to withdraw the nft.
When users withdraw all token amounts using `withdraw()`, it withdraws the nft. But if the nft is corrupted and impossible to transfer, `withdraw()` will revert.

- https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/NFTBoostVault.sol#L248

```solidity
    if (registration.withdrawn == registration.amount) {
        if (registration.tokenAddress != address(0) && registration.tokenId != 0) {
            _withdrawNft();
        }
        // delete registration. tokenId and token address already set to 0 in _withdrawNft()
        registration.amount = 0;
        registration.latestVotingPower = 0;
        registration.withdrawn = 0;
        registration.delegatee = address(0);
    }
```

### [L-04] `updateNft()` should check if users update with the same nft

`updateNft()` should prevent to update with the same nft to remove unnecessary withdrawal/deposit.

- https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/NFTBoostVault.sol#L305

```solidity
    function updateNft(uint128 newTokenId, address newTokenAddress) external override nonReentrant {
        if (newTokenAddress == address(0) || newTokenId == 0) revert NBV_InvalidNft(newTokenAddress, newTokenId);
```

### [L-05] `setMultiplier()` should check `multiplierValue >= 1e3` incase of positive value
It's not recommended setting a multiplier less than `1e3` because users can get that value without locking any nfts.

- https://github.com/code-423n4/2023-07-arcade/blob/88dcbdedebc506284fcfb3f14d20fc789ce811cf/contracts/NFTBoostVault.sol#L363

```solidity
    function setMultiplier(address tokenAddress, uint128 tokenId, uint128 multiplierValue) public override onlyManager {
```