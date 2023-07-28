I'm a new comer on auditing so I'm just testing out the auditing tools and the auditing process, Judges you can ignore this analysis as I consider is to basic.

I used Fuzzing test tools like echidna, make static testing with slither and symbolic analysis with manticore.
This is my report:

#### Issue ID: 001

**Title:** Ignored Return Value from Transfer Function

**Location:** NFTBoostVault.sol, Function `_lockTokens(address,uint256,address,uint128,uint128)`

**Description:** The `transferFrom` function call in the `_lockTokens` function does not check the return value. This could be a potential issue because if the `transferFrom` function fails for any reason, the `_lockTokens` function will still continue executing. If the `transferFrom` function doesn't revert on failure, this could lead to an inconsistent state within the contract.

**Potential Impact:** An attacker might exploit this to manipulate the state of the contract. If the `transferFrom` call fails silently, the contract could lock tokens that were not actually transferred, leading to an incorrect state.

**Severity:** Medium

**Recommendation:** Handle the return value of `transferFrom` and revert the transaction if it's `false`. This ensures that the contract state remains consistent with the actual token transfers.

## Issue ID: 002

**Title:** Sending Ether to Arbitrary Addresses

**Locations:**

- ReputationBadge.sol, Function `withdrawFees(address)`
- ArcadeTreasury.sol, Function `_spend(address,uint256,address,uint256)`

**Description:** In both the `withdrawFees` and `_spend` functions, Ether is being sent to an arbitrary address. This could be exploited if an attacker can gain control of the recipient address or if the recipient address is determined based on user input or contract state that can be manipulated by an attacker.

**Potential Impact:** An attacker could potentially divert funds to an address of their choice, leading to a loss of Ether for the contract.

**Severity:** High

**Recommendation:** Ensure that these functions can only be called by authorized addresses, and add checks to prevent the recipient address from being manipulated by potential attackers.

## Issue ID: 003

**Title:** Ignored Return Value from Transfer Function

**Location:** NFTBoostVault.sol, Function `_lockTokens(address,uint256,address,uint128,uint128)` at Line 657

**Description:** The `transferFrom` function call in the `_lockTokens` function does not check the return value. This could be a potential issue because if the `transferFrom` function fails for any reason, the `_lockTokens` function will still continue executing. If the `transferFrom` function doesn't revert on failure, this could lead to an inconsistent state within the contract.

**Potential Impact:** An attacker might exploit this to manipulate the state of the contract. If the `transferFrom` call fails silently, the contract could lock tokens that were not actually transferred, leading to an incorrect state.

**Severity:** Medium

**Recommendation:** Handle the return value of `transferFrom` and revert the transaction if it's `false`. This ensures that the contract state remains consistent with the actual token transfers.

### Time spent:
4 hours