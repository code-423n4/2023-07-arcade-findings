
# Information

## [I-01] Possible minting with zero amount
It is possible to mint with zero amounts. It will pass all checks and `mintPrice` will be 0

This check will also pass if the right arguments are set.
`if (!_verifyClaim(recipient, tokenId, totalClaimable, merkleProof)) revert RB_InvalidMerkleProof();`

This is not a crucial action and will not affect the protocol negatively."

```solidity
    function mint(
        address recipient,
        uint256 tokenId,
        uint256 amount,
        uint256 totalClaimable,
        bytes32[] calldata merkleProof
    ) external payable {
        //@audit what if amount is = 0

        // if token id = invalid, price = 0
        uint256 mintPrice = mintPrices[tokenId] * amount;
        uint48 claimExpiration = claimExpirations[tokenId];
```


## [I-02] Emit event at a crucial place.
It is good practice to emit an event at a crucial place. Setting the token is one of the most important things for the `ArcadeTokenDistributor` contract.

```solidity
 function setToken(IArcadeToken _arcadeToken) external onlyOwner {
        if (address(_arcadeToken) == address(0)) revert AT_ZeroAddress("arcadeToken");
        if (address(arcadeToken) != address(0)) revert AT_TokenAlreadySet();

        arcadeToken = _arcadeToken; //@note can emit event here
    }
```


## [I-03] Add additional check for rewardsRoot
Add additional check for `rewardsRoot` to be different than `bytes32(0)`;

```solidity
function setMerkleRoot(bytes32 _merkleRoot) external onlyOwner {
        rewardsRoot = _merkleRoot; //@note bytes32

        emit SetMerkleRoot(_merkleRoot);
    }
```

## [I-03] Minter and InitialDistribution addresses should be different
The Minter and InitialDistribution addresses should be different. It is good to have a check for that."

```solidity
    constructor(address _minter, address _initialDistribution) ERC20("Arcade", "ARCD") ERC20Permit("Arcade") {
        if (_minter == address(0)) revert AT_ZeroAddress("minter");
        if (_initialDistribution == address(0)) revert AT_ZeroAddress("initialDistribution");
        //@audit minter != _initialDistribution ?

        minter = _minter;

        mintingAllowedAfter = block.timestamp + MIN_TIME_BETWEEN_MINTS;

        // mint the initial amount of tokens for distribution
        _mint(_initialDistribution, INITIAL_MINT_AMOUNT); // 100_000_000 * 1e18
    }
```