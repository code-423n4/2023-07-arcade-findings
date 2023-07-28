## GAS-1: <X> += <Y> costs more gas than <X> = <X> + <Y> for state variables

### Description

Using the addition operator instead of plus-equals saves gas.

### Affected file

* ArcadeTreasury.sol (Line: 117, 198)
* ReputationBadge.sol (Line: 116)

## GAS-2: Divisions which do not divide by -X can be unchecked

### Description

Divisions with a positive value cannot underflow or overflow

### Affected file

* ARCDVestingVault.sol (Line: 330)
* ArcadeToken.sol (Line: 154)
* NFTBoostVault.sol (Line: 494, 633)

## GAS-3: Do not calculate constants

### Description

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

### Affected file

* ArcadeTreasury.sol (Line: 48, 49, 50)
* ReputationBadge.sol (Line: 44, 45, 46)

## GAS-4: Not using the named return variables when a function returns, wastes deployment gas

### Description

It is not necessary to have both a named return and a return statement.

### Affected file

* ARCDVestingVault.sol (Line: 381)
* BaseVotingVault.sol (Line: 148, 159, 170)

## GAS-5: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* ArcadeToken.sol (Line: 167)
* BaseVotingVault.sol (Line: 187, 196)
* NFTBoostVault.sol (Line: 701)

## GAS-6: Should use arguments instead of state variable

### Description

Using function's parameter cost less gas then a state variable.

### Affected file

* ArcadeToken.sol (Line: 136)

## GAS-7: Unbounded gas consumption risk due to external call recipients

### Description

In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using ```addr.call{gas: }```. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

### Affected file

* ArcadeTreasury.sol (Line: 341)

## GAS-8: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* ARCDVestingVault.sol (Line: 201)
* ArcadeAirdrop.sol (Line: 66)
* NFTBoostVault.sol (Line: 557, 657, 674)
* ReputationBadge.sol (Line: 167)

## GAS-9: Using > 0 costs more gas than != 0 when used on a uint in a require() statement

### Description

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance.

### Affected file

* NFTBoostVault.sol (Line: 589, 589)

## GAS-10: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* ArcadeTreasury.sol (Line: 341)