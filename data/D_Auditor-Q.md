### [LOW-01] Use block.number - 1 in arcadeTreasury::_spend() for reliability
It is a common practice to use block.number - 1 because the current block and its information isn't deterministic as long as the block isnt mined yet. current block expenditure should be recorded in the previous block and the limit could be checked against that. This way, the protocol is safe from the lack of determinism with current block heights.

Recommendation:
```solidity

        uint256 spentThisBlock = blockExpenditure[block.number - 1];
        if (amount + spentThisBlock > limit) revert T_BlockSpendLimit();
        blockExpenditure[block.number - 1] = amount + spentThisBlock;

```