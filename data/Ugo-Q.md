## Low issue: The new minter can be the old minter

In [ArcadeToken.sol](https://github.com/code-423n4/2023-07-arcade/blob/main/contracts/token/ArcadeToken.sol) in the `setMinter` method, if the address of the new minter is the same one with the old minter, a strange MinterUpdated event will be raised with no change in the minter address [here](https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132). This will be confusing.

#### Proposed solution:
It can be avoided by simply checking the equality of the 2 addresses and returning an error if the newMinter is the same as the old one:
```
if (_newMinter == minter) revert AT_SameMinter("newMinter");
```