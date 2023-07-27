The tokenURI function in BadgeDescriptor.sol would still return an uri even if there is no token created for the provided tokenId.
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/BadgeDescriptor.sol#L48
```
  function tokenURI(uint256 tokenId) external view override returns (string memory) {
        return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
    }
```
For example, if 0 is supplied as tokenId, the function would return a baseURI for a token that does not exist. This could cause confusion to users.