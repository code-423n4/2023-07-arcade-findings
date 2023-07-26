# First

It is more efficient, for `++X` and `X++` to do `X += 1`. See the next test in foundry to show that it saves 8 gas and 2 gas respectively:

```
pragma solidity ^0.8.13;

import "forge-std/Test.sol";

contract POC is Test {
    uint256 private a;

    function setUp() public {
        a = 0;
    }

    function testA() public {
        a++;
    }

    function testB() public {
        a += 1;
    }

    function testC() public {
        ++a;
    }
}
```

Results:

- testA -> 22397
- testB -> 22389
- testC -> 22391

Consider using the `X += 1` version everywhere. The RE can be `[^ \+]*\+\+` for the occurrences of `X++` and `\+\+[^ \)]*` for the occurrences of `++X`. The same applies to other arithmetic operations.

# Second

In loops where the number of runs is know in advance, update the counter to an `unchecked` block:

```
for(uint256 cont = 0; cont < ARRAY.length; ) {
    ....
