### Any comments for the judge to contextualize your findings
- Finding edge cases in vault contracts, with a special focus on NFTBoostVault.sol exploring scenarios where multiplier might be mis-accounted in vote counts.
- Finding edge cases in ArcadeTreasury.sol that lead to loss of funds.
- The particularity of core voting and GSC voting process and how the two processes might lead to unintended behaviors or loss of funds in ArcadeTreasury.sol.
- Finding general vulnerabilities that might lead to unintended behaviors or disruption in governance actions.
- Explore attack scenarios that might lead to DOS or loss of funds.

### Approach taken in evaluating the codebase
- Manual code walkthrough
- Custom scripts

### Architecture recommendations
Consider creating direct interactions between NFT contracts and NFTBoostVault.sol. 

Current contract implementations in ReputationBadge.sol and NFTBoostVault.sol segregate each other. For example, an NFT can be minted in ReputationBadge.sol for a user. But if this NFT is qualified for voting power boost, a user has to interact with NFTBoostVault.sol to add the NFT and delegate. 

Instead, consider allowing ReputationBadge.sol to interact with NFTBoostVault.sol to add NFT for users who have voting power but no linked NFT. 

In addition, consider allowing multiple NFT boosts for one registration in NFTBoostVault.sol. This will allow multiple NFTs to be added to the vault, further increasing the interactions between NFT and Vault contracts, at the same time, increasing user experience.

### Codebase quality analysis
- Sufficient contract and function documentation.
- Good test coverage.
- Occasional wasteful gas operation observed.
- Occasional unnecessary complexity in on-chain storage structure:
In ArcadeTreasury.sol, `blockExpenditure` is a mapping that stores the amount of token spent and approved every time with current block.number as the key. 
This mapping variable stores historical block.number with a corresponding value in storage. However, there is currently no on-chain query on historical values, nor is there event emitted with block.number to allow off-chain accounting to take advantage of the mapping structure.
Consider either removing the mapping variable and only storing the current block value, or if there is a need for historical values, re-write event to allow emmitting with block.number as event args so that proper off-chain accounting and queries can be done.

### Centralization risks
Low

### Mechanism review
In ArcadeTreasury.sol, some code might have overlapping functionalities under certain conditions.
The distinction between `smallSpend()`, `mediumSpend()` and `largeSpend()` might not be as clear when governance can use `largeSpend()` to spend a small amount. See details in the reports. 

### Systemic risks

In ArcadeTreasury.sol, ERC20 `approve()` is used with every governance or GSC-related approval action. There are known attack scenarios with ERC20 approve function. See details in the reports.






### Time spent:
25 hours