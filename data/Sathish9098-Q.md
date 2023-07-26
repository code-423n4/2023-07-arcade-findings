
# LOW FINDINGS

##

##

## [L-1] 

## [L-1] Lack of same value input control in critical setMinter() functions

###
The same address value should be avoided 

### POC

```
FILE: 2023-07-arcade/contracts/token/ArcadeToken.sol

 function setMinter(address _newMinter) external onlyMinter {
        if (_newMinter == address(0)) revert AT_ZeroAddress("newMinter");

        minter = _newMinter;
        emit MinterUpdated(minter);
    }


```
https://github.com/code-423n4/2023-07-arcade/blob/f8ac4e7c4fdea559b73d9dd5606f618d4e6c73cd/contracts/token/ArcadeToken.sol#L132-L137

### Recommended Mitigation

Add same value input control 

```
require(_newMinter!=minter, "Same address value ")
```





