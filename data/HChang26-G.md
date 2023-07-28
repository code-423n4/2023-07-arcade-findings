Arcade Gas:
```
Gas01: <x> += <y> costs more gas than <x> = <x> + <y> for state variables-same for <x> -= <y> .

```

```
Gas02: for loop length should be cached instead of reading from storage every time. ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow.
```

```
Gas03: Public function can be marked as external.
```

```
Gas04: Some public state variables can be marked as private.
```

```
Gas05: `mint()` in ArcadeToken.sol can be simplified by removing `if (_to == address(0)) revert AT_ZeroAddress("to”);`. This check is unnecessary and will automatically revert on ERC20.
```

```
Gas06: Do not emit `arcadeToken` in  ArcadeTokenDistributor.sol. It is already stored in state variable.
```

```
Gas07:  state variables in ArcadeTokenDistributor.sol can be re-position to save gas. Sort all bool variables next to each other.
```

```
Gas08: Functions guaranteed to revert when called by normal users can be marked payable.
```

```
Gas09: Consider removing string variable in custom error revert(). Avoiding the use of string arguments in the revert() function, as string operations can be computationally expensive and consume more gas compared to using error codes or custom errors.
```

```
Gas10: `_getWithdrawableAmount()` in NFTBoostVault.sol. Return block can be unchecked since `amount` is always greater than `withdrawn`.
            Unchecked{
		   return registration.amount - registration.withdrawn;
	    }
```
