## Issues found:
 G001: Don't Initialize Variables with Default Value

## Vulnerabilities:
  ../2023-07-arcade/contracts/ArcadeTreasury.sol::339 => for (uint256 i = 0; i < targets.length; ++i) {
  ../2023-07-arcade/contracts/NFTBoostVault.sol::345 => for (uint256 i = 0; i < userAddresses.length; ++i) {
  ../2023-07-arcade/contracts/nft/ReputationBadge.sol::144 => for (uint256 i = 0; i < _claimData.length; i++) {

## Impact
Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecessary gas. If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). If you explicitly initialize it with its default value, you are just wasting gas.

## Mitigations:
```
High Gas Usage
uint256 x = 0;
bool y = false;
```

```
Gas Usage Optimized
uint256 x;
bool y;
```
```
Another example
uint256 hello = 0; //bad, expensive
uint256 world; //good, cheap
```

## Issues found:
 G002 - Cache Array Length Outside of Loop
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

## Vulnerabilities:
 G002:
  ../2023-07-arcade/contracts/ArcadeTreasury.sol::337 => if (targets.length != calldatas.length) revert T_ArrayLengthMismatch();
  ../2023-07-arcade/contracts/ArcadeTreasury.sol::339 => for (uint256 i = 0; i < targets.length; ++i) {
  ../2023-07-arcade/contracts/NFTBoostVault.sol::343 => if (userAddresses.length > 50) revert NBV_ArrayTooManyElements();
  ../2023-07-arcade/contracts/NFTBoostVault.sol::345 => for (uint256 i = 0; i < userAddresses.length; ++i) {
  ../2023-07-arcade/contracts/errors/Treasury.sol::33 => * @notice Array lengths must match.
  ../2023-07-arcade/contracts/nft/BadgeDescriptor.sol::49 => return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";
  ../2023-07-arcade/contracts/nft/ReputationBadge.sol::141 => if (_claimData.length == 0) revert RB_NoClaimData();
  ../2023-07-arcade/contracts/nft/ReputationBadge.sol::142 => if (_claimData.length > 50) revert RB_ArrayTooLarge();
  ../2023-07-arcade/contracts/nft/ReputationBadge.sol::144 => for (uint256 i = 0; i < _claimData.length; i++) {

## [G002] Impact
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.
In this case where let's take line 144, ReputationBadge as an example, the context remains for others.

Iterates over a loop by using  ias an index. It performs the following operations:

Reading the length property of the _claimData array in every iteration which involves a state-read operation with the EVM.
Calling the function of the balance on the _vault, which could potentially be a state-changing operation or an external function call, both of which require a significant amount of gas.
Writing the output of that function call to the _claimData array, a potentially state-changing operation that uses more gas.

For the first point, in Ethereum, all data is stored in a data area called storage, which is persistent between function calls and transactions. Reading data from this area is expensive in terms of gas. If the length of the _claimData array is being fetched from storage directly in every iteration, it will invoke a SLOAD operation with the EVM, which is costly.

In this optimized version of the example, we avoid the costly SLOAD operation in every loop iteration and replace it with a MLOAD, which is only fractionally as expensive. That being said, we reduced the gas cost within the loop, which will result in substantial savings when the array length is substantial.

## [G002] Mitigation
```
Higher Gas Usage
for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
```
```
Lower Gas Usage
uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}
```

## Issues found:
 G003 - Use != 0 instead of > 0 for Unsigned Integer Comparison
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.

## Vulnerabilities:
 G003:
  ../2023-07-arcade/contracts/NFTBoostVault.sol::589 => if (change > 0) {
  ../2023-07-arcade/contracts/nft/BadgeDescriptor.sol::49 => return bytes(baseURI).length > 0 ? string(abi.encodePacked(baseURI, tokenId.toString())) : "";

## [G003] Impact
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.
In this case where let's take line 589, NFTBoostVault as an example, the context remains for others. `change` cannot be a negative value (as it's an unsigned integer), the conditions `change > 0` and `change != 0` would behave the same. Both would check if `change ` is non-zero. Changing the condition will not affect the behavior of the contract in this case. However, if `change` could be negative, the condition `change != 0` would trigger even if `change` is less than 0, while `change > 0` would only trigger if `change ` is greater than 0.

## [G003] Proof of Concept
```
Higher Gas Usage
// `a` being of type unsigned integer
require(a > 0, "!a > 0");
```
```
Lower Gas Usage
// `a` being of type unsigned integer
require(a != 0, "!a > 0");
```

