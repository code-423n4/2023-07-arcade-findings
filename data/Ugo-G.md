## Low issue: Token address can be changed multiple times despite having a comment saying the opposite

In [ArcadeTokenDistributor.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeTokenDistributor.sol) in the method `setToken` the comment says:
>"The token contract address can only be set once

but there is no technical restriction in the code to call it multiple times by the owner and so to change the token address multiple times.

#### Proposed solution:
The set should happen in the constructor if it should be set only once or the comment should be removed since it can be set multiple times.


## Low issue: ArcadeAirdrop contract inherits Authorizable but it is not needed

The contract [ArcadeAirdrop](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeAirdrop.sol) inherits the OpenZeppelin Authorizable contract even if it only use the `onlyOwner` modifier. This is not necessary since there are several methods in the Authorizable contract that are not needed for that contract.

#### Proposed solution:
`ArcadeAirdrop.sol` just needs to inherit the OpenZeppelin Ownable contract. It would be cheaper to deploy.


## Low issue: The event SetDescriptor shares the same name with a method of the contract

The contract [ReputationBadge](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/nft/ReputationBadge.sol) contains an event called [SetDescriptor](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L61), the name is the same with a [method in that contract](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/nft/ReputationBadge.sol#L184). This brings confusion to the contract and is not coherent with the other events names.

#### Proposed solution:
Rename this event into `DescriptorSet` to be coherent with the names of the other events of this contract (`RootsPublished`, `FeesWithdrawn`)


## Low issue: The blockExpenditure map stores the spendings and approvals in the same block while the comments only mention spending

In the contract [ArcadeTreasury](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/ArcadeTreasury.sol) the method [_approve](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/ArcadeTreasury.sol#L384) saves the number of tokens approved in the current block. It seems that the number of tokens spent and approved in the same block are added together for no reason. 

#### Proposed solution:
Only the spent tokens should be added to keep track of how much is actually spent in the block.

