## [L-01] SWC-120 Potential use of "block.number" as source of randonmness.

## Impact
```txt
The environment variable "block.number" looks like it might be used as a source of randomness. 
Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. 
Also keep in mind that attackers know hashes of earlier blocks. 
Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
```

## Proof Of Concept
Vulnerable Code
```sol
// https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L362
        blockExpenditure[block.number] = amount + spentThisBlock;

// https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L360
        uint256 spentThisBlock = blockExpenditure[block.number];
```

## Tools Used
```txt
Mythx 
VS Code
```

## Remediation
```txt
Using external sources of randomness via oracles, and cryptographically checking the outcome of the oracle on-chain. e.g. Chainlink VRF. This approach does not rely on trusting the oracle, as a falsly generated random number will be rejected by the on-chain portion of the system.
Using commitment scheme, e.g. RANDAO.
Using external sources of randomness via oracles, e.g. Oraclize. Note that this approach requires trusting in oracle, thus it may be reasonable to use multiple oracles.
Using Bitcoin block hashes, as they are more expensive to mine.
```

## [L-02] SWC-120 Potential use of "block.number" as source of randonmness.

## Impact
```txt
The environment variable "block.number" looks like it might be used as a source of randomness. 
Note that the values of variables like coinbase, gaslimit, block number and timestamp are predictable and can be manipulated by a malicious miner. 
Also keep in mind that attackers know hashes of earlier blocks. 
Don't use any of those environment variables as sources of randomness and be aware that use of these variables introduces a certain level of trust into miners.
```

## Proof Of Concept
Vulnerable Code
```sol
// https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L106
            startTime = uint128(block.number);

// https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L234
        if (grant.cliff > block.number) revert AVV_CliffNotReached(grant.cliff);

// https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L319
        if (block.number < grant.cliff) {

// https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L323
        if (block.number >= grant.expiration) {

// https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ARCDVestingVault.sol#L328
        uint256 blocksElapsedSinceCliff = block.number - grant.cliff;
```

## Tools Used
```txt
Mythx 
VS Code
```

## Remediation
```txt
Using external sources of randomness via oracles, and cryptographically checking the outcome of the oracle on-chain. e.g. Chainlink VRF. This approach does not rely on trusting the oracle, as a falsly generated random number will be rejected by the on-chain portion of the system.
Using commitment scheme, e.g. RANDAO.
Using external sources of randomness via oracles, e.g. Oraclize. Note that this approach requires trusting in oracle, thus it may be reasonable to use multiple oracles.
Using Bitcoin block hashes, as they are more expensive to mine.
```