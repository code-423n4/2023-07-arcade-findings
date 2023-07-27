[ReputationBadge.sol#L171]("https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L171")

## Impact
If the recipient is a contract with a complex receive() function, the gas will exceed 2300 and an out-of-gas revert will occur.

## Recommended Mitigation Steps

Check the recipient address, 
```solidity
    require(address(recipient).code.length == 0, "");    
```
or use a call function and limit gas.
```solidity
    address(recipient).call{gas: gasLimit, value: balance}("");
```